import time, csv, os, sys, logging
from tqdm import tqdm, trange
import pprint
from py2neo import Node, Relationship, Graph, NodeMatcher
from selenium import webdriver
from selenium.webdriver.chrome.webdriver import WebDriver
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.wait import WebDriverWait
from selenium.webdriver.common.by import By
from selenium.common.exceptions import NoSuchElementException
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.chrome.options import Options

log = logging.getLogger(__name__)
graph = Graph("bolt://localhost:7687",auth= ("neo4j","PASSWORD"))
tx = graph.begin()

# Helper Functions
# ------------------------------------------bra------------------------------------

def createDriverInstance():
    capabilities = {
        "chromeOptions": {
            'args': ['--start-maximized', '--disable-infobars', '--disable-notifications',  "user-agent=Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36",'--disable-extensions', '--disable-gpu', '--no-sandbox', '--user-data-dir=/Users/samfinton/Documents/Programming/Frigg/Default']
        }
    }
    driver = webdriver.Chrome(desired_capabilities=capabilities)
    return driver

def infiniteScroll(driver):
    print("Running infiniteScroll")
    bottomOfList = driver.find_elements_by_xpath('//*[@id="medley_header_photos"]/a')
    while len(bottomOfList) < 1:
        bottomOfList = driver.find_elements_by_xpath('//*[@id="medley_header_photos"]/a')
        driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
        time.sleep(0.3)
    else:
        print("Scrolled to end of friends list")


def wait(amount):
    print("Waiting...")
    time.sleep(amount)

# ------------------------------------------------------------------------------

def scrape_Friends(driver, url):
    current_URL = url + "/friends"
    print("Scraping Friends on:", current_URL)
    driver.get(current_URL)
    wait(.3)
    infiniteScroll(driver)
    Root_FriendsList = {}
    just_friend_links = []
    elems = driver.find_elements_by_tag_name("a")
    for elem in elems:
        try:
            elem_HREF = elem.get_attribute("href")
            if elem_HREF is None:
                continue
            else:
                if elem_HREF.endswith('/friends'):
                    parentDiv = elem.find_element_by_xpath("..")
                    elem_NAME = parentDiv.find_element_by_css_selector("a").get_attribute("innerText")
                    elem_HREF = elem_HREF[:-8]
                    #Root_FriendsList.update({elem_NAME: elem_HREF})
                    just_friend_links.append(elem_HREF)
        except:
            pass
    pprint.pprint(just_friend_links)
    with open('input_test.txt', 'w') as f:
        for item in just_friend_links:
            f.write("%s\n" % item)
    return Root_FriendsList

def scrape_Info(driver,url):
    print("Scrape Info started on:", url)
    aboutInfo = {}
    section = ["/about?section=education", "/about?section=living","/about?section=contact-info", "/about?section=relationship", "/about?section=bio"]
   # For each page in about section
    for i in section:
        currentSection = url + i
        driver.get(currentSection)
        driver_Wait = WebDriverWait(driver, 15).until(EC.presence_of_element_located((By.XPATH, "//*[contains(@id,'pagelet_timeline_app_collection_')]/ul/li/div/div[2]/div/div")))
        sectionDiv = driver.find_element_by_xpath("//*[contains(@id,'pagelet_timeline_app_collection_')]/ul/li/div/div[2]/div/div")
        sectionText = sectionDiv.get_attribute("innerText").splitlines()
        keywordFilter = ["No"]
        original_List = list(filter(None, sectionText))
        clean_List = [str for str in original_List if not any(i in str for i in keywordFilter)]
        dict_Title = str(clean_List[0])
        length = len(clean_List)%2
        if length != 0:
            aboutInfo[dict_Title] = dict(clean_List[i:i+2] for i in range(1, len(clean_List), 2))
        else:
            clean_List.pop(0)
            aboutInfo[dict_Title] = dict(clean_List[i:i+2] for i in range(1, len(clean_List), 2))

    return aboutInfo
    print(url,"'s about section scraped")


def create_Root_Node(name, url, aboutInfo):
    labels = "Person"
    properties = {"name":name, "URL":url}

    for outerKey, outerValue in aboutInfo.items(): #NOT WORKING
        for innerKey, innerValue in outerValue.items():
            properties.update({innerKey: innerValue})
    root_Node = Node(*labels, **properties)
    if NodeExistence(name) == False:
        print("Root Node being created")
        tx.create(root_Node)
        #time.sleep(1)
        #tx.commit()
    else:
        print("Root Node already exists")
    return root_Node

def create_Node(name, url, aboutInfo):
    labels = "Person"
    properties = {"name":name, "URL":url}

    for outerKey, outerValue in aboutInfo.items():
        for innerKey, innerValue in outerValue.items():
            properties.update({innerKey: innerValue})
    pprint.pprint(properties)
    currentNode = Node(*labels, **properties)
    if NodeExistence(name) == False:
        print("Node being created")
        tx.create(currentNode)
        #time.sleep(1)
        #tx.commit()
    else:
        print("Node already exists")
    return currentNode


    # graph.create(Relationship(a, "FRIENDS_WITH", b))

def NodeExistence(name):
    matcher = NodeMatcher(graph)
    m = matcher.match("Person", name=name).first()
    if m is None:
        return False
    else:
        return True

if __name__ == "__main__":

    # for eachFriend in user1.friendList
    #       query neo4j nodes with username
    #       if username already exists as node
    #           user1 = FRIENDS_WITH existing_userNode
    #       else
    #           create node for eachFriend containing name:"" and userURL:""
    #           go to userURL / about
    #           scrape aboutInfo and append to users respective node
    #           go to userURL / friends
    #           scrape usersFriends and append to users respective node
    #

    root_Name = "Sam Finton"
    root_URL = "https://www.facebook.com/Sam.finton"
    final_shutdown = True
    try:
        #Create driver instance
        driver = createDriverInstance()
        #Scrape root info
        #aboutInfo = scrape_Info(driver, root_URL)
        # Scrape root's friends list
        scrape_Friends(driver, root_URL)
        #Create root node (if it doesn't already exist)
        #root_Node = create_Root_Node(root_Name, root_URL, aboutInfo)

        # for name, url in Root_FriendsList.items():
        #     #Scrape user info
        #     aboutInfo = scrape_Info(driver, url)
        #     pprint.pprint(aboutInfo)
        #     #Scrape user's friends
        #     friends_List = scrape_Friends(driver, url)
        #     pprint.pprint(friends_List)
        #     #Create new node for each name (if it doesn't already exist)
        #     currentNode = create_Node(name, url, aboutInfo)
        #     #Create relationship between currentNode and root_Node
        #     #graph.create(Relationship(root_Node, "FRIENDS_WITH", currentNode))


    except Exception as e:
        final_shutdown = False
        log.exception("Exception Shutdown:")
        print(e)
        driver.quit()
        sys.exit()

    finally:
        if final_shutdown == True:
            print("Finally quit")
            driver.quit()
            sys.exit()
