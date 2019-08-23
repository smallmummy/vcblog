---
layout:     post
title:      The Experience of Scraping Instagram(Crawler)
subtitle:   Crawler
author:     vincent_c
categories: [ crawler, python ]
image: assets/images/crawler.jpg
catalog: true
tags: [Crawler, featured,stick_post]
---  
There are several articles on the internet to teach you to scrape data from many websites. However, this is not a tutorial on how to scrape data from Instagram step by step. There are summarized experiences in this article from the work which  I tried to scrape data from Instagram. Hope you can skip some traps that were recorded in this article when next time you do it.

The idea which drove me to do this was from an article on Wechat which described how to scrape massive poem data and analyze them, finally output a diagram shown a relationship between those poets. At that time, I had the idea to do similar things on SNS, and output some figures which can show the relationship among people, and then I chose Instagram as the target. There is one more reason why I decided to choose Instagram: the considerable pictures in that can make me practice some AI project, like machine learning and image identification.

## About the Legal and Ethical Aspects of Data Scraping
This is a hard question. We need to consider this behavior from many aspects. I defend for myself as:

1. Scraping in itself is not illegal.
1. The data I scraped is from Instagram public page, which everyone can access.
1. The method I scraped the data is based on Selenium, which emulate the real Chrome browser behaviour. It won't increase load to server.

More discussion can refer to:

* [Legal and Ethical Aspects of Data Scraping](https://www.datahen.com/legal-ethical-aspects-data-scraping/)
* [Is scraping and collect Instagram data legal? Is it legal due to the contractual terms?](https://www.quora.com/Is-scraping-and-collect-Instagram-data-legal-Is-it-legal-due-to-the-contractual-terms)
* [美国数据爬虫相关案例判决梳理](https://www.secrss.com/articles/9746)

![](https://cl.ly/2f367714176c/web_scraping.png)

## 1. Introduction
First of all, I have to mention this is not the proper timing for us to scrape massive data from Instagram, and there are the reasons:

1. Instagram had been deprecating old API platform since 2018. Although official announcement said that the basic API will keep on until early 2020, however, the basic API is not useful as I think.

	Ps: the API of fetching Followers and Following don't belong to basic API and was cancelled early on.

	Here is the screenshot from Instagram:
	
	![](https://cl.ly/0a6c3faa77ac/Image%2525202019-07-24%252520at%2525204.42.17%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)
	<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
	Figure: screenshot from Instragram</div></center>

2. Please notice the “huge data” in my preceding mention. If your target is just downloaded few pictures or comment of a specific user or like some individual posts automatically in real-time, then there are several codes which can perform your requirement in Github, like a bot. But if you aim to scrape massive data from huge users including their Followers or Following or their posts, that’s colossal data which require you seek a smart and efficient method to get under the current situation.

At last, I have to remind you I am just a beginner who enter this area recently, what I record in this article is just my personal experience, I know it looks stupid, and maybe not suitable and useful for you. However, if you have an excellent idea or you could correct some mistake in this article, please comment below, and I will appreciate that.

## 2. Ideas and Flow

### 2.1 Method and Tools

* Method:

    Since the API is not available, I consider using request and response HTTP to get the data. I choose the Selenium as the tools, and the advantages are apparent: it could help me emulate every operation on Chrome, and I needn't consider and analyze the cookie and some encoding data. I, a blank paper on JS area, decide within a nanosecond.
    
    However, the disadvantages are apparent as well: the efficiency, under the concerning of unavailable API, I couldn't seek a better way by my perspective. one more is the prestige of Selenium: MEMORY HUNGER BEAST!
	
    The method is straightforward: using Selenium to emulate the real operation in a browser, then fetch the response data from Selenium, extract the target data and keep it.

* Tools:

	Python and Selenium
	
### 2.2 Flow:

![](https://cl.ly/0834dd309d85/ins%252520crawler%252520flow.png)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
	Figure: Flow illustration</div></center>


#### 2.2.1 Main Frame

**Process:**

1. Guard process in the crontab will check periodically, and start the crawler if there is no survived one.
1. Take a task from task pool on cloud(such as crawl user profile or data for followers and followed, or user posts)
1. Execute the task.
1. Check whether the implementation of the task is accomplished. And perform some post-process for the uncomplete task.
1. Dump the crawled data into a file on the local machine.
1. Update the flag of the task in task pool to completed or partial or failed(there will be another program to process the incomplete or failed task later)
1. Add more task in task pool if needs. For instance, after finish crawl all the follower of user A, then you can put all user A's follower in task pool, and crawl their information later.
1. Check whether up to the threshold, if already reached, then release the resource and quit. If not yet, continue to the next loop.
	
**Note:**

* The guard process can restart the crawler actively if AWS kill the crawler due to some reason.
* Setting a task pool on the cloud can provide the mechanism of distribution process by different machines.
* There are several reasons could cause the halt of task execution, like time out, or lag of loading due to a slow network, etc. the task executer will dump the data as much as possible until the end of crawled data, for not wasting the effort previously. however, I need to set a flag to let me process the unsuccessful task later.
* The threshold that makes the crawled quit at a specific occasion could make the crawler survived longer. Because Selenium would consume more and more resource along the time of execution, so the mechanism that quit for a while and pulled by crontab is a better way.
	
#### 2.2.2 Crawl the user profile
This is the easiest part among the three crawling (profile, Followers & Following and posts) because all you want is in the page source:    

![](https://cl.ly/188f6ac71467/%2525E5%2525B1%25258F%2525E5%2525B9%252595%2525E5%2525BF%2525AB%2525E7%252585%2525A7%2525202019-07-25%252520%2525E4%2525B8%25258B%2525E5%25258D%2525882.35.44.png)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
	Figure: content in page source</div></center>
	
You can quickly locate using Regular Express in one part of javascript in page source. 
There is too much information, including, but not limited to:

* Biography
* Avatar
* Whether private
* Amount of Followers
* Amount of Following
* Brief of several posts/videos
* etc

**Brief Process:**

1. Login
1. Jump to the link: www.instagram.com/user_name
1. Fetch page source
1. Parse the Json
1. Dump the data



**Note:**

* It's not necessary to log in before you crawl the user profile in the page source. As long as the account is not private, all the user profile has been loaded in page source after the link "www.instagram.com/user_name" is loaded. If that's private account, and that account didn't validate your application of following, the information you crawled in the user profile is limited, even if you logged in.
* Since I use the MongoDB, I dump all the Json into DB after fetch data from page source without any process on data. I will add the logic into another program to process those data if needs.

#### 2.2.3 Crawl the Follow & Followedby of users

As I plan to crawl the data by emulating the real operation in Chrome, I thought the process is not complicated (however later it proved I'm wrong), and it likes the following GIF shown:

![](https://cl.ly/66e5bd7e5371/Screen%252520Recording%2525202019-07-25%252520at%25252003.02%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.gif)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
	Figure: browse Follow/Followedby data manually</div></center>

**Brief Process:**

1. Login
1. Crawl user profile
1. Click the button "Follower" or "Following"
1. Wait for the new window popup
1. Scroll down to the bottom
1. Wait for loading new data
1. Repeat until all the data had been loaded
1. Parst all data in page source
1. Dump the data


**Note:**

* You have to login first, and then you are allowed to have a look at another user's Follower & Following. If that's a private account and it didn't validate your application of following, you cannot crawl any data on it.
* Anyway I have to login first and forward to his/her main page like "www.instagram.com/user_name", then I can crawl the data of Follower & Following. Hence I simply put the function of crawl user profile into crawl Follower & Following.

#### 2.2.4 Crawl the posts of users

Same with above, we need to check how the browser works, then we can emulate it.

There are two ways I found and both work:

1. Go the main page of the user, click the thumb of 1st post, then crawl all the data in the new popup window, then close it. Click the thumb of 2nd post and repeat.
1. Same with above, but after crawling all the data of 1st post, click the arrow on the right indicated "the previous post" instead of close the popup windows.

I found the 2nd method is secure, and the real operation likes the following GIF shown:

![](https://cl.ly/65bbcee5a6d7/Screen%252520Recording%2525202019-07-25%252520at%25252003.22%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.gif)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
	Figure: browse posts data manually</div></center>

**Brief Process:**

1. Login
1. Crawl user profile
1. Cilck the 1st post
1. Wait for the new popup window
1. Crawl all the data including picture URL, list of liker and all the comments.
1. Click the button of arrow on the right side
1. Wait for loading new post
1. Repeat until crawled all posts
1. Dump the data

**Note:**

* I am still doing on this part, so I will supply the detail after it's accomplished.

## 3. Detail of Program

Until now, the contents listed above like the simple frame and there are some question in our brain, like:

* How do you know you already scroll down to the bottom?
* How do you handle the time out of loading?
* How do you know the new data was loaded?
* Why do you need to parse page source instead of detail of web element from Selenium?
* etc.

Some questions confuse me a long time, and some still are not solved by now. I will demonstrate the detail in this chapter.

### 3.1 Login

I thought this part is simple until Instagram request to verify my account as Instagram treat my account as suspicious account, then I have to add the manual validation process.

**the flow chart:**

![](https://cl.ly/d825e1d6c1c4/Image%2525202019-07-25%252520at%2525204.09.14%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)

<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
Figure: flow of Login</div></center>

**Code for login:**

```
@retry(attempt=5, wait=1)
def login(self):
    browser = self.browser
    url = "%s/accounts/login/" % (InsCrawler.URL)
    try:
        browser.get(url)
    except selenium.common.exceptions.TimeoutException:
        self.logger.warn("loading login page failed, retry...")
        raise RetryException()

    u_input = browser.find_one('input[name="username"]')
    p_input = browser.find_one('input[name="password"]')

    self.logger.info("ready to log in")
    u_input.send_keys(secret.username)
    p_input.send_keys(secret.password)

    login_btn = browser.find_one(".L3NKy")
    login_btn.click()

    @retry(attempt=3, wait=5)
    def check_login():
        if browser.find_one('input[name="username"]'):
            raise RetryException()
        else:
            # sometimes instagram require you input the code when they think it's necessary to verify your account
            if browser.current_url.find(InsCrawler.VERIFY_URL) != -1:
                # use hook to pass the verification
                self.logger.info("found ins request verify the account, now hook the code....")
                self.HookToPassVerification()
            elif browser.current_url != "https://www.instagram.com/":
                self.logger.warn(
                    "the link after login({0}) might be wrong, please check".format(browser.current_url))
                raise RetryException()

    check_login()
    self.logger.info("log in successfully!")
```

**Code for pass verification:**

```
def HookToPassVerification(self):
    try:
        @retry(attempt=10, wait=1)
        def wait4loading():
            ele_follower_frame = self.browser.find_one("#security_code")

            # It takes time to load the post for some users with slow network
            if ele_follower_frame is None:
                # print("trying...")
                self.logger.warn("the frame of input code is not loaded yet, retry...")
                raise RetryException()

        @retry(attempt=12, wait=20)
        def wait4VerificationCode():
            if not os.path.exists(InsCrawler.HOOK_FILE):
                self.logger.info("there is no hook file:{0},cannot hook,retry...".format(InsCrawler.HOOK_FILE))
                raise RetryException()
            else:
                fs = open(InsCrawler.HOOK_FILE, "r")
                hook_code = fs.read().strip()
                fs.close()

                if hook_code == "":
                    self.logger.info(
                        "there is no code in hook file:{0},waiting for the code....".format(InsCrawler.HOOK_FILE))
                    raise RetryException()
                else:
                    return hook_code

        @retry(attempt=3, wait=5)
        def wait4login():
            if self.browser.current_url != "https://www.instagram.com/":
                self.logger.warn(
                    "the link after verified({0}) might be wrong, please check".format(self.browser.current_url))
                raise RetryException()

            self.logger.info("logged in!")

        self.logger.debug("begin to verify!")
        btn_send_code = self.browser.find_one("#react-root > section > div > div > div > form > span > button")

        if btn_send_code is None:
            self.logger.error("didnt get the button for request verification code,need check the page source")
            self.logger.debug(writeFile.writeFile("login", self.browser.driver.page_source, "html"))
            raise LackElementException()

        self.logger.debug("click the button to request code!")
        btn_send_code.click()

        time.sleep(2)
        wait4loading()

        self.logger.debug("verification page was shown,now waiting for the code from HOOK FILE...")
        verification_code = wait4VerificationCode()

        self.logger.debug("got the code:{0}".format(verification_code))
        code_input = self.browser.find_one("#security_code")

        if code_input is None:
            self.logger.error("didnt get the input box for verification code,need check the page source")
            self.logger.debug(writeFile.writeFile("login", self.browser.driver.page_source, "html"))
            raise LackElementException()

        code_input.send_keys(verification_code)

        btn_submitCode = self.browser.find_one('#react-root > section > div > div > div > form > span > button')

        if btn_submitCode is None:
            self.logger.error("didnt get the button for submit verification code,need check the page source")
            self.logger.debug(writeFile.writeFile("login", self.browser.driver.page_source, "html"))
            raise LackElementException()

        self.logger.debug("now subit the code....")
        btn_submitCode.click()

        self.logger.debug("waiting for refreshing.....")
        wait4login()

    finally:
        self.logger.debug("empty the hook file...")
        fs = open(InsCrawler.HOOK_FILE, "w")
        fs.write("")
        fs.close()
```


**Why Instagram assume it's suspicious account?**

I found it's nothing to do with whether you use Selenium or headless or not. Occasionally my program logs in the account using another HTTP proxy, then Instagram immediately send me an email and notify there is one-time suspicious login(Instagram is not accessible directly from China, and I mistakenly forgot to switch back the usual proxy.....)

If your program always uses the same fixed IP to operate, I think you can skip this step.


### 3.2 Crawl User Profile

**Code:**

```
# go to main page of profile
browser = self.browser
url = "%s/%s" % (InsCrawler.URL, username)
browser.get(url)

# crawl profile
source = browser.driver.page_source
p = re.compile(r"window._sharedData = (?P<json>.*?);</script>", re.DOTALL)
json_data = re.search(p, source).group("json")
data = json.loads(json_data)
user_data = data["entry_data"]["ProfilePage"][0]["graphql"]["user"]
```

### 3.3 Crawl Follow and Followedby

I attempted many different methods in this part, and optimize the efficiency continuously(however the current way which I used now still cannot satisfy me). I will write down all the methods here from the earliest one, hope the evolution will inspire you.

#### 3.3.1 scrolling and scraping (abandoned)
**Brief Process:**

1. Be aware the target amout of Followers/Following
1. Get all data in new popup windows
1. Check each whether already were saved or not
1. Save the new data
1. Scroll down
1. Repeat above procedures until you scrape all Followers data

**Flow Chart:**

![](https://cl.ly/6bd76da580ea/Image%2525202019-07-26%252520at%25252010.58.08%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
Figure: flow of scrolling and scraping</div></center>

**Illustration**

![](https://cl.ly/8b7ea19085e4/%2525E5%2525B1%25258F%2525E5%2525B9%252595%2525E5%2525BF%2525AB%2525E7%252585%2525A7%2525202019-07-26%252520%2525E4%2525B8%25258A%2525E5%25258D%25258811.14.20.png)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
Figure: how Instagram load new data</div></center>

What we need to get primarily is the frame of the popup window, then fetch all current data. In the first batch, there will be a list which consists 12 Follow data loaded firstly, then scroll down and get the new data, until all finish.

Firstly, I'm not sure which method new data loaded insert by:

* New data loaded will be inserted at the end by sequence or
* New data loaded will be mixed randomly

And I didn't think this will impact much at that time, and then I treat is as the second method.

**Code:**

```
# until get all data for Follow
while len(posts) < num_follower and wait_time < TIMEOUT:
    post_num, wait_time = start_fetching(pre_post_num, wait_time)
    pbar.update(post_num - pre_post_num)
    pre_post_num = post_num

    if wait_time > TIMEOUT / 2:
        break
```

**Code for function "start_fetching":**

```
def start_fetching(pre_post_num, wait_time):

    # get the fram of new popup window
    ele_followers = browser.find(".PZuss > li")  # headless

    # traverse all followers
    for ele in ele_followers:

        # get the id of follow
        ele_id_tmp = browser.find_one(".d7ByH a", ele)
        ele_id = ele_id_tmp.get_attribute("title")

        # check whether that id already save before
        if ele_id not in key_set:
            # 查找头像link
            ele_img = browser.find_one("._2dbep img", ele)
            img_url = ele_img.get_attribute("src")

            # save the new ID, for check whether dupilicated later
            key_set.add(ele_id)

            # dump all info
            posts.append({"key": ele_id, "img_url": img_url})

        # if the total amount fetched in this time as same as the previous one, that indicates loading failed, no new data loaded(need retry)
    if pre_post_num == len(posts):
        pbar.set_description("Wait for %s sec" % (wait_time))
        sleep(wait_time)
        pbar.set_description("fetching")

        wait_time *= 2
        browser.scroll_up(300)
    else:
        wait_time = 1

    pre_post_num = len(posts)

    s_num = len(ele_followers)
    ele_followers[s_num - 1].location_once_scrolled_into_view
    
    return pre_post_num, wait_time
```

**Explanation:**

This code (return an array) could get all Followers/Following data including their account and avatar link.

```
ele_followers = browser.find(".PZuss > li")
```

Then traverse the content, skip the ID which already was saved before and keep the new ones.

The following code can extend the wait time when the browser cannot load the new data due to the slow network.

```
if pre_post_num == len(posts):
    pbar.set_description("Wait for %s sec" % (wait_time))
    sleep(wait_time)
    pbar.set_description("fetching")

    wait_time *= 2
    browser.scroll_up(300)
else:
    wait_time = 1
```

I use the following method to perform the scrolling down: choose the last element of Followers/Following which was placed the bottom of the frame and was not displayed in the view. Then scroll it into view. Meanwhile, Instagram will load new data.

```
s_num = len(ele_followers)
ele_followers[s_num - 1].location_once_scrolled_into_view
```

**Advantage:**

Simple and don't need to consider the complicated situation

**Disadvantage:**

Inefficiency. Indeed the above code works when the amount of Follow is not too much, like below 1K. But the traversal of all Follow data for checking the duplicated data ate much time, and it's not necessary.

**Reflection:**

After I tested several times, I found that the new data loaded will be supplemented at last by sequence, namely neither mixed nor randomly.

I think from another side of the view about this issue: it also will be convenient for the web developer to add the new loaded data by sequence. However I hadn't been a web developer in my career, maybe I'm wrong, the reversed of preceding situation might exist.

Then I consider about how if I repeat the scrolling down until all the data loaded displayed in that frame, then I perform one-off fetching all the data at the last step.

That method will skip the judgment process for checking duplicated data, and then it could become efficient. Here is it below.


#### 3.3.2 scrolled, then one-off scraping (abandoned)

**Brief Process:**

1. Check how many data were loaded in frame
1. Scroll down
1. Repert above two steps until the amount of loaded data equals that of Followers/Following
1. Fetch all data loaded

**Flow Chart:**

![](https://cl.ly/9fa8aa796b2e/Image%2525202019-07-26%252520at%2525202.24.57%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
Figure: Flow of scrolled then one-off scraping</div></center>

**Code:**

```
isAll, cur_num = scroll_down_all(num_follower)
start_fetching()
return isAll, posts
```

**Code for scrolling down until all loaded**

```
@retry(attempt=5, wait=1)
def scroll_down_all(total_num):
    s_num = 0
    # 之前一共取了多少个
    pre_num = 0

    while s_num < total_num - 1:

        # 检查当前页面一共加载了多少个
        ele_followers = browser.find(".PZuss > li")  # headless
        s_num = len(ele_followers)

        # 如果和上一次加载的一样多，表示没有加载新的
        if s_num == pre_num:
            wait_time *= 1.5

            # 如果超时
            if wait_time > TIMEOUT:
                self.logger.error(
                    "loading fans detail timeout, the data is not completed,just get {0}/{1} items".format(
                        s_num, total_num))
                return False, s_num
        else:
            wait_time = 0.6

        pre_num = s_num
        self.logger.debug("getting fans {0}/{1}...".format(s_num, total_num))
        
        ele_followers[s_num - 1].location_once_scrolled_into_view

        '''
        # 向下滚动，加载所有的follower
        actions = ActionChains(browser.driver)
        actions.move_to_element(ele_follower_frame)
		actions.send_keys_to_element(ele_follower_frame,Keys.END)
        actions.perform()
        '''

        time.sleep(wait_time)

    self.logger.debug("drag all pages for all fans")
    return True, s_num
```
**Code for one-off fetching all data:**

```
def start_fetching():
    # get all data in the frame
    ele_followers = browser.find(".PZuss > li")

    self.logger.debug("found {0} fans in this page".format(len(ele_followers)))

    isComplete = True

    # begin to travering
    for i, ele in enumerate(ele_followers):
        # 找到follower的id
        ele_id_tmp = browser.find_one(".d7ByH a", ele)
        if ele_id_tmp is None:
            self.logger.warn("the {0}th d7ByH didnt exist in this page!".format(i))
            isComplete = False
            continue

        ele_id = ele_id_tmp.get_attribute("title")
        if ele_id is None:
            self.logger.warn("the {0}th title didnt exist in this page!".format(i))
            isComplete = False
            continue



        # save all the data of Follow
        posts.append({"id": username, "followedby": ele_id})
```

**Explanation:**

As you found, there is another method to perform scrolling down, and I commented that because I found ActionChains works inefficiently when the data in that frame more than 6K (it's common that some users have more than 10K followers).

The method ActionChains is simple as well, it emulates the browser operations: choose the frame which you need scroll down, then send key of "DOWN" to perform the scrolling down:

```
actions = ActionChains(browser.driver)
actions.move_to_element(ele_follower_frame)
actions.send_keys_to_element(ele_follower_frame,Keys.END)
actions.perform()
```

Until now, it works well and can crawl many user data, until the program threw the TIME OUT exception when it was trying to scrape the data loaded more than 2400.

#### 3.3.3 Time out Issue

(the TIME OUT caused by the slow network was not in the scope of our discussion, my current solution is to supply enough exception catch and retry)

The detail in the log is not explicit, which just state "timeout":

```
2019-07-15 19:33:00,936 - crawler.py[line:712] - ERROR: Traceback (most recent call last):
  File "/home/ubuntu/vc_code/ins/inscrawler/crawler.py", line 600, in get_follower_detail
    start_fetching()
  File "/home/ubuntu/vc_code/ins/inscrawler/crawler.py", line 482, in start_fetching
    ele_id = ele_id_tmp.get_attribute("title")
  File "/home/ubuntu/.local/lib/python3.6/site-packages/selenium/webdriver/remote/webelement.py", line 141, in get_attribute
    self, name)
  File "/home/ubuntu/.local/lib/python3.6/site-packages/selenium/webdriver/remote/webdriver.py", line 627, in execute_script
    'args': converted_args})['value']
  File "/home/ubuntu/.local/lib/python3.6/site-packages/selenium/webdriver/remote/webdriver.py", line 312, in execute
    self.error_handler.check_response(response)
  File "/home/ubuntu/.local/lib/python3.6/site-packages/selenium/webdriver/remote/errorhandler.py", line 242, in check_response
    raise exception_class(message, screen, stacktrace)
selenium.common.exceptions.TimeoutException: Message: timeout
  (Session info: headless chrome=75.0.3770.100)
```

To reproduce the error, I have to switch headless mode to head Chrome, and check the response in the tab of "Network", and I found:

![](https://cl.ly/79b4a44e7920/%2525E5%2525B1%25258F%2525E5%2525B9%252595%2525E5%2525BF%2525AB%2525E7%252585%2525A7%2525202019-07-26%252520%2525E4%2525B8%25258B%2525E5%25258D%2525883.24.27.png)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
Figure: Tab "network" in Chrome Develop Mode</div></center>

Before I use 0.8 seconds of interval waiting time for each scrolling down, after each scrolling down, another new 12 elements will be loaded, namely, totally invoke 200 times API for loading (12*200 = 2,400).

Instagram set the rate limit, even for normal browser, I found that in Instagram official website:

![](https://cl.ly/4cdebe203853/Image%2525202019-07-26%252520at%2525203.49.34%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
Figure: Rate limit from official</div></center>

That 200 limit is for Instagram Graph API, well, I don't think the rate limit for normal browsing is 200 per hour because after I change the interval waiting time to 4 seconds, my program finished scraping 6K data no more than 35 minutes:

```
6,000 / 12(per API invoke) = 500 times
```

Regarding the rate limit, I will discuss it later. Here I share a method on how to locate TIME OUT issue under Chrome Development Mode.


#### 3.3.4 Effiency Issue

Due to the rate limit from Instagram, after I increase the interval waiting time, the program runs well. But along the more and more user Follow data(more than 6K), the efficiency issue became more and more distinct and crucial. After I check the log,  I found it consume more than 2 hours to scrape 6K Follow data.

And I don't think it works if I feed more memory to Selenium, as the CloudWatch provided by AWS showed, seems there is still available memory(Swap). However, Selenium runs extremely slow.

![](https://cl.ly/aea9ba55d68e/%2525E5%2525B1%25258F%2525E5%2525B9%252595%2525E5%2525BF%2525AB%2525E7%252585%2525A7%2525202019-07-26%252520%2525E4%2525B8%25258B%2525E5%25258D%2525884.23.58.png)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
Figure: AWS CloudWatch</div></center>

After I reviewed my code again, I found there are many places that need to be improved if I obey these following principles:

**Principle One: reduce the unnecessary Selenium opertaion like find element**

I early on use 3 Selenium operation in one iteration:
![](https://cl.ly/7bbacb1ba3d9/%2525E5%2525B1%25258F%2525E5%2525B9%252595%2525E5%2525BF%2525AB%2525E7%252585%2525A7%2525202019-07-26%252520%2525E4%2525B8%25258B%2525E5%25258D%2525885.53.25.png)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
Figure: 3 Selenium operation in one iteration</div></center>

1. Locate the frame, then I know how much data were already loaded, which can be used for checking whether all Follow data were loaded.
1. Locate the last item for scrolling down later
1. Scroll down the last item to let Instagram load new data.

For reducing the operation, I change the logic to:

1. Locate the last item, then I could get the text of it
1. Scroll the last item
1. If the current text of the item is the same as the previous one, I could assume no new data was loaded due to slow network or already reached the last item

**Key code:**

```
# select the last item using xpath
ele = browser.find_xpath("/html/body/div[3]/div/div[2]/ul/div/li[last()]")

if ele.text == pre_text:
    self.logger.debug("the last element name is still {0}".format(pre_text))
    wait_time *= 2
else:
    wait_time = init_wait_time
    pre_text = ele.text

# scroll into view for loading new items
ele.location_once_scrolled_into_view
```

**Principle Two: JS > Xpath > CSS when atempt to locate(in my case)**

I used CSS selector to locate in the early stage, but I found Xpath is better than CSS selector when I was trying to locate items in the XML response. then I changed the function below from

```
def find(self, css_selector, elem=None, waittime=0):
    obj = elem or self.driver
    if waittime:
        WebDriverWait(obj, waittime).until(
            EC.presence_of_element_located((By.CSS_SELECTOR, css_selector))
        )
    try:
        return obj.find_elements(By.CSS_SELECTOR, css_selector)
    except NoSuchElementException:
        return None
```

change to 

```
return obj.find_element_by_xpath(css_selector)
```

Actually, find element by ID or Name is more efficient than Xpath and CSS selector, but in my case, I found that there is no name nor ID in the items which I attempt to scrape.

**Performance Contrast**

Each scrolling down could load 12 items, then if load 6K data completely, need around 500 items scrolling down, namely reload. in the log, I found the performance plummet down after around 300~400 times reload in the previous solution.

the performance figure:
![](https://cl.ly/8bdbf9696b5a/Image%2525202019-07-26%252520at%2525205.11.55%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
Figure: Performance Contrast</div></center>

As mentioned above, when you try to locate or operate, the performance of JS is better than others.

So I have to change the code again, use JS instead of Xpath.

```
ele_last_temp = browser.find_xpath("/html/body/div[3]/div/div[2]/ul/div/li[last()]")

browser.driver.execute_script('arguments[0].scrollIntoView(true);', ele_last_temp)

last_text_temp = ele_last_temp._id
```

**The performance after the change:**

![](https://cl.ly/c7146ef11a47/Image%2525202019-07-26%252520at%2525208.44.20%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
Figure: Performance Contrast After Optimization</div></center>

Then I found there barely is lag when performing Selenium operation.

#### 3.3.5 JS ScrollTo + Height(current solution)

The primary reason which makes me drop using select element plus scroll into view is Selenium always throw this exception when it runs a long time: 
> selenium.common.exceptions.StaleElementReferenceException: Message: stale element reference: element is not attached to the page document

Several reasons would cause that exception, and I will describe it in the later chapter.

What I want is scroll the content in the popup window, and make Instagram reload the new content, when we do it in the Chrome, as long as our mouse is in the scope with the popup window, we don't have to click or select a specific element, why we have to use the method "Scrollintoview"?

**Flow Chart:**

![](https://cl.ly/0739160783f8/Image%2525202019-07-26%252520at%2525209.12.31%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
Figure: Flow Chart</div></center>

**Code:**

```
# locate the frame first
refresh_frame()

cur_height = 0
# each time when the current height is not changed will extend the wait time
while wait_time < WAIT_TIME_OUT:

    scroll_element(cur_height)
    cur_height = browser.driver.execute_script('return arguments[0].scrollHeight;',self.ele_frame_popup)

    if cur_height == pre_height:
        wait_time *= 2
    else:
        wait_time = init_wait_time
        pre_height = cur_height
```

**Code for scroll down:**

```
def scroll_element(tmp_cur_height):
    try:
        tmp_script = "arguments[0].scrollTo(0,{0})".format(int(tmp_cur_height) + 552)
        browser.driver.execute_script(tmp_script, self.ele_frame_popup)
        time.sleep(wait_time)
        return
    except:
        refresh_frame()
        raise RetryException()
```

Usually, the following code will execute once, however, if Selenium throws the exception like "stale element" due to some reasons, the new element will be  located again by the following code:

```
def refresh_frame():
    self.ele_frame_popup = browser.find_xpath("/html/body/div[3]/div/div[2]")
    if self.ele_frame_popup is None:
        self.logger.debug("the frame of followedby is None")
        self.logger.debug(writeFile.writeFile(username, browser.driver.page_source, "html"))
        return None
    return self.ele_frame_popup
```

**About the Height:**

There is a trick in the new popup window which would confuse you about the real height:


![](https://cl.ly/620508d12b1a/%2525E5%2525B1%25258F%2525E5%2525B9%252595%2525E5%2525BF%2525AB%2525E7%252585%2525A7%2525202019-07-27%252520%2525E4%2525B8%25258A%2525E5%25258D%2525888.46.59.png)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
Figure: Illustration about Height</div></center>


As you can see from the above figure, the class "PZuss" is the element where store the information of  Follow or Follower; however, it cannot be scrolled down. I think there is no attribute in it.

So we need to scroll down the element "isgrP" which is the parent node of "PZuss". But under "isgrP", there are more than one elements, and the height return by the following JS is not the real height:

```
document.getElementsByClassName("isgrP")[0].scrollHeight
> 2587
```

It's evident if you use Chrome Develop mode to inspect that you will find the mainframe which can be scrolled down consists of 3 parts:

![](https://cl.ly/af445b92cb11/1111.png)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
Figure: competed content under the frame</div></center>

As the above shown, what we need is just the first part, and the real height is 552, the return height as 2587 is the total height which includes the part that Instagram suggests the people you may know.

The problem is, if you scroll down too fast, namely you scroll down to the 3rd part over the end of 1st part which we actual need, Instagram will not reload and the window will hang there, then failed.

And the solution is simple as well: you need scroll to the end of 1st part which class name is "PZuss", which require you to calculate every time.

The height of each sub-item inside is 46, Instagram will reload 12 more items each time, that means each time you need scroll to 552 from the current position.


After I tried in Chrome and found only the initial height is not the real one. After you scroll down once, Instagram will take back the 3rd part, which suggests people you may know, and then the height return from 2nd time is real and safe.

![](https://cl.ly/0dff73873320/%2525E5%2525B1%25258F%2525E5%2525B9%252595%2525E5%2525BF%2525AB%2525E7%252585%2525A7%2525202019-07-23%252520%2525E4%2525B8%25258B%2525E5%25258D%25258812.03.25.png)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
Figure: debug the height issue</div></center>


#### 3.3.6 Fetch data by find_element or page source

Since I separate the reloading/scrolling data and fetch data, then I consider the right part shown in the below figure:

![](https://cl.ly/3b262438b549/Image%2525202019-07-27%252520at%2525209.56.38%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
Figure: Structure</div></center>

In the beginning, I still use the original way to fetch all the data which were already loaded in Selenium: locate the frame again, then Selenium will return all data under that frame, and then what left need I do is elements traversal for fetching.

```
ele_followers = browser.find("body > div.RnEpo.Yx5HN > div > div.isgrP > ul > div > li")  # headless
for i, ele in enumerate(ele_followers):

    ele_id_tmp = browser.find_one("div > div > div > div > a", ele)

    if ele_id_tmp is None:
        self.logger.warn("the {0}th d7ByH didnt exist in this page!".format(i))

    ele_id = ele_id_tmp.get_attribute("title")
    if ele_id is None:
        self.logger.warn("the {0}th title didnt exist in this page!".format(i))
```


It worked when elements are small, when the amount of elements is more than 6K, this method likes the way you are trying to push an elephant into a fridge, it makes me crazy.

After I calm down, I found I'm stupid. Everything in the universe has its destiny, namely what you should do and shouldn't do. find_element in Selenium is just for locating element, and indeed you can get the attributes from the element after you locate it, but that's an only auxiliary function. I review what's in my hand, and suddenly realize what I need is Regular Express which can help me fetch the data by some pattern rapidly because all the data were already loaded!


```
page_source = browser.driver.page_source
all_fans = re.findall('title="([a-zA-Z0-9_\.]*)"\s+href="', page_source)

count_followedby_got = len(all_fans)

for i in range(count_followedby_got):
    posts.append({"id": username, "followedby": all_fans[i]})v
```

The above code runs within 1 second and returns all I wanted.


### 3.4 Crawl All Posts

This part is still in the stage of performance optimization. Therefore I described here briefly.

**Brief Flow:**

1. Login
1. Click the 1st(lastest post) and wait for it popup
1. Fetch all data
1. (not close popup window)Click the button the right
1. Wait Instagram load the previous post
1. Repeat the above three steps until it finishes

**Code:**

```
def _get_posts_full(self, num):
    @retry()
    def check_next_post(cur_key):
        ele_a_datetime = browser.find_one(".eo2As .c-Yi7")

        # It takes time to load the post for some users with slow network
        if ele_a_datetime is None:
            raise RetryException()

        next_key = ele_a_datetime.get_attribute("href")
        if cur_key == next_key:
            raise RetryException()

    browser = self.browser
    browser.implicitly_wait(1)
    ele_post = browser.find_one(".v1Nh3 a")
    ele_post.click()
    dict_posts = {}
    
    cur_key = None
    # Fetching all posts
    for _ in range(num):
        dict_post = {}

        # Fetching post detail
        try:
            check_next_post(cur_key)

            # Fetching datetime and url as key
            ele_a_datetime = browser.find_one(".eo2As .c-Yi7")
            cur_key = ele_a_datetime.get_attribute("href")
            dict_post["key"] = cur_key
            fetch_datetime(browser, dict_post)
            fetch_imgs(browser, dict_post)
            fetch_likes_plays(browser, dict_post)
            fetch_likers(browser, dict_post)
            fetch_caption(browser, dict_post)
            fetch_comments(browser, dict_post)

        except RetryException:
			pass
        except Exception:
			pass
            traceback.print_exc()

        dict_posts[browser.current_url] = dict_post
        left_arrow = browser.find_one(".HBoOv")
        if left_arrow:
            left_arrow.click()

    posts = list(dict_posts.values())
    if posts:
        posts.sort(key=lambda post: post["datetime"], reverse=True)
    return posts
```


## 4. Tips

### 4.1 Methods for Scroll Down
Nowadays, more and more site reload the content dynamically, which need you to scroll down the page and make the browse reload the new data. Unless you are talent in JS, it's tough to send the POST which needs you to analyze the JS and simulate it to request reloading data.

#### 4.1.1 using ActionChains
It can simulate your real operation in the browser, and the sample code is:

```
from selenium import webdriver
from selenium.webdriver.common.action_chains import ActionChains

driver = webdriver.Chrome()
driver.get('http://www.w3schools.com/')
target = driver.find_element_by_link_text('BROWSE TEMPLATES')
actions = ActionChains(driver)
actions.move_to_element(target)
actions.perform()
```

#### 4.1.2 scrollIntoView in JS
You should locate the element first.

```
driver = webdriver.Chrome()
driver.get('http://www.w3schools.com/')
target = driver.find_element_by_link_text('BROWSE TEMPLATES')
driver.execute_script('arguments[0].scrollIntoView(true);', target)
```

#### 4.1.3 element into view under element
Similar with above, you should locate the element first:

```
driver = webdriver.Chrome()
driver.get('http://www.w3schools.com/')
target = driver.find_element_by_link_text('BROWSE TEMPLATES')
target.location_once_scrolled_into_view
```

#### 4.1.4 send keys under element
Simulate pressing the "Down" key to scroll.

```
driver = webdriver.Chrome()
driver.get('http://www.w3schools.com/')
driver.find_element_by_tag_name('body').send_keys(Keys.END) # Use send_keys(Keys.HOME) to scroll up to the top of page
```

#### 4.1.5 scrollTo in JS
If you want to scroll the main page instead of some particular popup window, it will be straightforward and efficient, as it lacks a step of locating element:

```
driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
```

Regarding the popup window, you have to locate it first, then use "scrollTo":

```
tmp_script = "arguments[0].scrollTo(0,{0})".format("Y")
browser.driver.execute_script(tmp_script, self.ele_frame_popup)
```

#### 4.1.6 My Experience

Without the consideration of performance, all methods above worked well. If there is a performance requirement for you, I recommend using ScrollTo in JS.

Although both ScrollIntoView and ScrollTo require you locate the element first, after I tested, ScrollIntoView will have a slower speed after Selenium runs a long time.

One more, don't locate element frequently, it will impact performance much. Just locate necessarily.

#### 4.2 Error of "StaleElementReferenceException: Message: Element not found"

It's evident from the literal meaning: after you locate an element which will be assigned _id as unique identification, however when you want to use it, this ID is stale, out of the expiry.

Here is the official explanation:
> Possible causes of StaleElementReferenceException include, but not limited to:
> 
> * You are no longer on the same page, or the page may have refreshed since the element was located.
> * The element may have been removed and re-added to the screen, since it was located. Such as an element being relocated. This can happen typically with a javascript framework when values are updated and the node is rebuilt.
> * Element may have been inside an iframe or another context which was refreshed.

The actual reason and solution depends on your real situation, and you can

* Refresh the whole page(will impact much)
* Operate it immediately after you locate it
* Re-locate it if it became stale.


Usually in my code, I prefer using:

* Locate the element for the first time
* Operate it or fetch attribute
* Catch the exception as StaleElementReferenceException
* Re-locate the element again if that exception was thrown.

### 4.3 Locate Element
there are several methods provided by Selenium to locate element:

> * find_element_by_id
> * find_element_by_name
> * find_element_by_xpath
> * find_element_by_link_text
> * find_element_by_partial_link_text
> * find_element_by_tag_name
> * find_element_by_class_name
> * find_element_by_css_selector

#### 4.3.1 Efficiency and Priority

Base on my actual experience, I list them below as priority which I prefer:

1. by_id
1. by_name/by_class_name/by_tag_name
1. by_xpath
1. by_css_selector
1. and so on

If the ID or Name is existed and unique, by ID or name is efficient and straightforward. However, I always found there is no name, or the name will be changed sometimes in my practical scene.

XPath or CSS selector is similar, but I prefer to use Xpath when I tried to find something in XML response, and there are several functions in Xpath are powerful.

#### 4.3.2 How to get the expression in Xpath or CSS
it's convenient if you use Chrome Develope mode to get it:

"Open Chrome Develope Mode" -> click button "select an element in the page to inspect it" at the left top  -> select your target element -> go to right window and click the three dots on left of your element code -> in the popup menu choose copy, then select Xpath or CSS

**the GIF:**

![](https://cl.ly/d7dc4d364827/Screen%252520Recording%2525202019-07-27%252520at%25252011.47%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.gif)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
Figure: locate element under the Chrome develop mode</div></center>

### 4.4 the impact to the server due to the head/headless or Chrome Develope Mode 

#### 4.4.1 server response differently due to the head or headless Chrome

During my coding, I found my code works in head mode Chrome because I debug my code under the head mode Chrome. But it doesn't work anymore after I switch to headless mode, to be exact, the code of locating element couldn't work well.

I found the impact sometimes is:

* Class name will be different or even null when in headless mode.
* Server will response some addition DIV layer or something else when in headless mode.

However, it depends on server behavior

![](https://cl.ly/9fdeed45e8a2/Image%2525202019-07-27%252520at%2525203.39.47%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
Figure: changes due to headless mode</div></center>

**My suggestions:**

1. Be cautious when you use the class name to locate element, sometimes you can use Xpath or CSS to locate it. Or add enough exception catching to increase your code robustness.
1. Perform sanity test in both head and headless mode(depends on your running environment)
1. When finally you need run your code in the remote server with headless mode, the develope and test sequence could be: develop/test in local with head mode -> develop/test in local with headless mode -> git to remote server and test.

#### 4.4.2 Develop Mode may change the frame structure

For instance, when I try to locate this button which could pop up the window display all data of Follow, I open the Develop Mode to locate (CSS selector) as

```
#react-root > section > main > div > ul > li:nth-child(2) > a > span
```


However, my code didn't work, after I fetched the page source and found the real CSS selector should be:

```
#react-root > section > main > div > header > section > ul > li:nth-child(2) > a > span
```

And when I test the code in head mode, it works. But after I open Develop Mode in the head mode for debugging something, I found it didn't work. then it can be proven that Develop Mode may change the frame structure

![](https://cl.ly/effb033cef49/Image%2525202019-07-27%252520at%2525203.52.03%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
Figure: Develop Mode changes the structure</div></center>

### 4.5 Struggle with the Hunger Memory Beast

I found Selenium consume the memory rapidly after around half to one hour. And 1G memory is insufficient for a single-process Selenium program (without Swap Memory), then I have to add 1G Swap Memory to feed this beast.

There are my experiences, and perhaps it could be useful for you.

* Add more memory, including SWAP space.
* Periodically quit the Selenium and restart it again, which could release the memory deeply. If the login was involved, set, and keep the cookies would be helpful.
* Use the headless mode.
* Use javascript instead of others if the real situation allows.
* Initial Selenium driver without loading Images.

    ```
    chromeOptions = webdriver.ChromeOptions()
    prefs = {'profile.managed_default_content_settings.images':2}
    chromeOptions.add_experimental_option("prefs", prefs)
    driver = webdriver.Chrome(chrome_options=chromeOptions)
    ```

* Scrape Websites using Disk Cache.

    ```
    chromeOptions = webdriver.ChromeOptions()
    prefs={"profile.managed_default_content_settings.images": 2, 'disk-cache-size': 4096 }
    chromeOptions.add_experimental_option('prefs', prefs)
    driver = webdriver.Chrome(chrome_options=chromeOptions)
    ```

* Close more functions in the options which it won't be used in your program's features. (notice: that depends on your real situation, 2 is the value of "Close". I think you can guess the options meaning base on the literal.)

    ```
    prefs = {
        "profile.default_content_setting_values.notifications": 2,
        "profile.managed_default_content_settings.stylesheets": 2,
        "profile.managed_default_content_settings.cookies": 2,
        "profile.managed_default_content_settings.javascript": 2,
        "profile.managed_default_content_settings.plugins": 2,
        "profile.managed_default_content_settings.popups": 2,
        "profile.managed_default_content_settings.geolocation": 2,
        "profile.managed_default_content_settings.media_stream": 2,
    }
    chrome_options.add_experimental_option("prefs", prefs)
    ```




