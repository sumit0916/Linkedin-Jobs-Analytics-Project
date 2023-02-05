# Scraping job ids from linkedin
    import requests
    import pandas as pd
    from bs4 import BeautifulSoup
    #considering jobs from top 10-12 metro cities of india only.(because we need only 300+ jobs)

    job_details=[]
    for i in range(20):
        url= f"https://www.linkedin.com/jobs/search?location=india&trk=homepage-jobseeker_recent-search&position=1&pageNum={i}"
        response=requests.get(url)
        soup=BeautifulSoup(response.content,'html.parser')
        result=soup.find_all('div', class_="base-card relative w-full hover:no-underline focus:no-underline base-card--link base-search-card base-search-card--link job-search-card")
    
    for out in result:
        jobid=out.get('data-entity-urn')
        i_d=[]
        for i in jobid:
            if i.isnumeric():
                i_d.append(i)
        jbid=''.join(i_d)
    
        job_details.append(int(jbid))
    #bangalore,mumbai,hyderabad,delhi,pune,noida,gurugram,ahmedabad,chennai,lucknow
    li=[105214831,106164952,105556991,105282602,103671728,104869687,104793846,104990346,115702354,102335936]
    for j in li:
        for i in range(20):
            url= f"https://www.linkedin.com/jobs/search?keywords=&location=India&locationId=&geoId=102713980&f_TPR=&f_PP={j}&position=1&pageNum={i}"
            response=requests.get(url)
            soup=BeautifulSoup(response.content,'html.parser')
            result=soup.find_all('div', class_="base-card relative w-full hover:no-underline focus:no-underline base-card--link base-search-card base-search-card--link job-search-card")
        
        for out in result:
            jobid=out.get('data-entity-urn')
            i_d=[]
            for i in jobid:
                if i.isnumeric():
                    i_d.append(i)
            jbid=''.join(i_d)
    
            job_details.append(int(jbid))
    len(job_details)
    len(set(job_details))
    job_ids=list(set(job_details))
    job_ids
    len(job_ids)
    df=pd.DataFrame(job_ids)
    df
    df.to_csv("jobid.csv")
    
    
# Python code to automate linedin login and scrape the data using job ids
       import pandas as pd
    #reading jobid csv file

    df=pd.read_csv("jobid.csv")
    df
    # creating a list of job id dataframe

    li=df.id.values.tolist()
    import pandas as pd
    from selenium import webdriver
    from selenium.webdriver.chrome.options import Options
    from selenium.webdriver.common.by import By
    from bs4 import BeautifulSoup
    import requests
    from time import sleep
    chrome_options = Options()
    chrome_options.add_experimental_option("detach",True)
    driver = webdriver.Chrome(options=chrome_options)
    path=r"D:\Win_32_driver\chromedriver_win32"
    driver.get('https://www.linkedin.com/login')
    sleep(3)
    # login into linkedln

    username=driver.find_element(By.ID,'username')
    username.send_keys('your email')                            ###################################################### <<<<<ADD EMAIL ID HERE 
    password=driver.find_element(By.ID,'password')
    password.send_keys('your password')                            ###################################################### <<<<<ADD PASSWORD HERE
    login_button=driver.find_element(By.CLASS_NAME,'btn__primary--large')
    login_button.click()
    sleep(5)
    project_data=[]
    # this loop will give final raw data , but loop execution will take a lot of time maybe more than 45 minutes, depends on length of li list.

    for i in li:
        try:
            driver.get(f'https://www.linkedin.com/jobs/view/{i}')
            sleep(2)
            driver.execute_script("window.scrollTo(0,document.body.scrollHeight)")
            sleep(2)

            soup=BeautifulSoup(driver.page_source,"html.parser")
            sleep(1)

            tit=soup.find('h1',class_="t-24 t-bold jobs-unified-top-card__job-title")
            titlee=tit.text.strip()

            com=soup.find('span',class_="jobs-unified-top-card__company-name")
            companyy=com.text.strip()

            loc=soup.find('span',class_="jobs-unified-top-card__bullet")
            locationn=loc.text.strip()

            workplace=soup.find('span',class_="jobs-unified-top-card__workplace-type")
            workplacee=workplace.text.strip()

            applicants=soup.find('span',class_='jobs-unified-top-card__subtitle-secondary-grouping t-black--light')
            applicantss=applicants.text.strip()

            involvement=soup.find('li',class_="jobs-unified-top-card__job-insight")
            involvementt=involvement.text.strip()

            followers=soup.find('div',class_='artdeco-entity-lockup__subtitle ember-view t-16')
            followerss=followers.text.strip()

            a=soup.find('div',class_="t-14 mt5")
            typee=a.text.strip()

            job_dict={
                'job_title':titlee,
                'company':companyy,
                'location':locationn,
                'workplace':workplacee,
                'applicants':applicantss,
                'involvement':involvementt,
                'followers':followerss,
                'industry and employees':typee
                }

            project_data.append(job_dict)
            sleep(2)
        except:
            continue

    driver.close()
    df=pd.DataFrame(project_data)
    df
    # converting dataframe into csv file
    df.to_csv("project_database.csv")


# SQL Queries for data insights

use unit_3;


select * from jobs;
select * from company;
select * from details;

-- Comparison of number of jobs across different cities for different level 
select  Location, level,count(jobid) as Num_of_jobs from jobs join details on jobs.detailid = details.detailid
group by level,location order by Num_of_jobs desc;

-- number of jobs distribution across various industry. For instance, what is the total number of jobs in Software Industry as compared to Marketing
select Industry, count(jobid) as Num_of_jobs from jobs join company on company.company_id = jobs.company_id
group by industry order by Num_of_jobs desc;

-- number of opening with respect to the current employee count
select employees, count(jobid) as Num_of_jobs from jobs join company on company.company_id = jobs.company_id
group by employees order by Num_of_jobs desc;

-- relation b/w a company's social media presence and number of openings
select company, followers, count(jobid) as Num_of_jobs from jobs join company on company.company_id = jobs.company_id
group by company, followers order by Num_of_jobs desc;

-- Jobs having the most number of openings
select job_title, count(jobid) as num_of_jobs from jobs group by job_title order by num_of_jobs desc;

-- Count the number of jobs across different industry across different locations. For instance, 
-- how many Frontend Engineer jobs are there in Bangalore as compared to Data Analytics jobs in Bangalore
select industry, location, count(jobid) as num_of_jobs from jobs join company on company.company_id = jobs.company_id
group by industry, location order by num_of_jobs desc;
[SQL Queries.txt](https://github.com/sumit0916/Linkedin-Jobs-Analytics-Project/files/10610548/SQL.Queries.txt)

