- 👋 Hi, I’m @AnkitMaltare
- 👀 I’m interested in Python programming
- 🌱 I’m currently learning Data Analytics
- 📫 How to reach me ankitmaltare@gmail.com

<!---
Hello everyone this is a small E-voting project created using Python and SQL queries, it has 2 parts one is for the admin and the other for voters.

Admin can create a campaign add Candidates to it or he can view results.
Voter can only vote.

functions same can be modified or its codes can be used for some other task. 
you will find a "How To E_vote.docx" which gives information about how to use this code and its.
--->

import pandas as pd
import numpy as np
import mysql.connector as c
from datetime import date, datetime
from tabulate import tabulate
import getpass
import tkinter as tk

# Drop Database if required:

SQL Connector for 
conn = c.connect(host= "localhost",
                        user='root',
                        password= getpass.getpass("enter SQL password: "))
cursor=conn.cursor()
cursor.execute('DROP DATABASE E_VOTING') 
print("     ")
print("     ")
print("Database Deleted......")

# Drop database ends here

# Database Creation in SQL Starts here it also inserts test data in the table just to check everything is working perfectly:

def DatabaseSQL():

#---------------------Create Database in SQL-----------------
    conn = c.connect(host= "localhost",
                          user='root',
                          password= getpass.getpass("enter password: "))
    cursor=conn.cursor()
    cursor.execute('CREATE DATABASE E_VOTING') 

    #--------------------Create Table Campaign, Candidate, Voter, Votes for admin SQL----------------
    conn = c.connect(host= "localhost",
                          user='root',
                          password= getpass.getpass("enter password: "),
                          database="E_VOTING")
    cursor=conn.cursor()
    cursor.execute("CREATE TABLE IF NOT EXISTS Campaign(Campaign_Id int auto_increment primary key,Campaign_Name varchar(255) NOT NULL,Start_Date datetime,End_Date datetime)")
    cursor.execute("CREATE TABLE IF NOT EXISTS Candidate(Candidate_Id int auto_increment primary key,Candidate_first_Name varchar(255) NOT NULL,Candidate_last_Name varchar(255) NOT NULL,Party_Name varchar(255),Campaign_Id int,FOREIGN KEY (Campaign_Id) REFERENCES Campaign(Campaign_Id))")
    cursor.execute("CREATE TABLE IF NOT EXISTS Voter(Voter_ID varchar(255) primary key,Voter_Name varchar(255))")
    cursor.execute("CREATE TABLE IF NOT EXISTS Votes(Campaign_Id int,Candidate_Id int,Voter_ID varchar(255),VoteDateTime date,FOREIGN KEY (Campaign_Id) REFERENCES Campaign(Campaign_Id),FOREIGN KEY (Candidate_Id) REFERENCES Candidate(Candidate_Id),FOREIGN KEY (Voter_ID) REFERENCES Voter(Voter_ID))")               

    #-------------------Insert Test Data in all 4 Table-------------
    #------------Campaign
    
    cursor=conn.cursor()
    sql =  'insert into Campaign(Campaign_Id,Campaign_Name,Start_Date,End_Date) VALUES (%s,%s,%s,%s)'
    val = (int(101),'test','2023-10-10','2023-10-10')
    cursor.execute(sql,val)
    conn.commit()
    print(cursor.rowcount, "record inserted.")
    #------------------Candidate
    
    cursor=conn.cursor()
    sql =  'insert into Candidate(Candidate_Id,Candidate_first_Name,Candidate_last_Name,Party_name,Campaign_Id) VALUES (%s,%s,%s,%s,%s)'
    val = (int(10001),'test','test','test',int(101))
    cursor.execute(sql,val)
    conn.commit()
    print(cursor.rowcount, "record inserted.")
    #-------------------Voter
 
    cursor=conn.cursor()
    sql =  'insert into Voter(Voter_ID,Voter_Name) VALUES (%s,%s)'
    val = ('SLM1254','test')
    cursor.execute(sql,val)
    conn.commit()
    print(cursor.rowcount, "record inserted.")

    #------------------Votes

    cursor=conn.cursor()
    sql =  'insert into Votes(Campaign_Id,Candidate_Id,Voter_ID,VoteDateTime) VALUES (%s,%s,%s,%s)'
    val = (int(101),int(10001),'SLM1254',datetime.now())
    cursor.execute(sql,val)
    conn.commit()
    print(cursor.rowcount, "record inserted.")

DatabaseSQL()

# Database creation Ends here

# Connect to database: 

conn = c.connect(host= "localhost",
                        user='root',
                        password=getpass.getpass("enter SQL password to Connect: "),
                        database='E_VOTING')
cursor=conn.cursor()
print("Connected Sucessfully")

# End

# Dummy Voter ID and password of voters who can vote(just to test). this is optional you can your own Voters into the database:

def Dummy_Pass():
    conn = c.connect(host= "localhost",
                    user='root',
                    password= getpass.getpass("enter password: "),
                    database='E_VOTING')
    play = pd.read_excel('VoteData.xlsx')
    print(play)
    for i,j in play.iterrows():
        cursor=conn.cursor()
        sql =  'insert into Voter(Voter_ID,Voter_Name) VALUES (%s,%s)'
        val = tuple(dict(j).values())
        cursor.execute(sql,val)
        conn.commit()
    print("Dummy Record Inserted.")
  
Dummy_Pass()    

# Below is the function which is the main part of the project after running the above codes successfully you will be able to run this part:

def Portal_login():
    print()
    print("Welcome To E-Voting Portal Please press\n\n 'A'- Admin Login \n \n 'V'- Voter Login")
    print("")
    Port_Login = (input()).upper()
    if Port_Login == 'A':
        print("=====================")
        print('|   Hello Admin   |')
        print("=====================")
        return Adminlogin()
    elif Port_Login == 'V':
        print("=====================")
        print('Hello Voter')
        print("=====================")
        return Voter_login()
    else:
        print('Wrong input Please Try Again')
        return Portal_login()
#------------- Admin-------

def Adminlogin(count = 0):
    count +=1
    if count <= 3:
        a = input("enter User ID: ")
        b = input("enter password: ")
        if a == 'a' and  b == 'b': 
            print('login Sucessful')
            return Admin_options()
        else:
            print('Wrong Id or Password',3-count,' attempt left..')
            return Adminlogin(count)            
    else:
        print('Login count exceede, ID-LOCKED')

def Admin_options():
    print()
    print('   Please Choose From Below options ')
    print('=====================================')
    Admin_Functions = pd.Series(data=['Ceate Campaign','Publish Reult'],index=['Press "A" to-->','Press "B" to-->'])
    display(Admin_Functions)
    print()
    print()
    Selection = (input()).upper()
    if (Selection == "A"):
        Create_Campaign()
    elif (Selection == "B"):    
        return Publish_Result() 
    else:
        print('###.............')
        print("Wrong Selection try again")
        print()
        Decs = (input("Press 'T' to Try Again or any key exit: ")).upper()
        if (Decs == "T"):
            return Admin_options()
        
        else:
            print("You Logged Out")
#----------------Campaign Creation---------
def Create_Campaign():
    print('|======================================|')
    print('    Please Enter Campaign Detail ')
    print('|======================================|')
    Campaign_Name = (input('Enter Campaign Name:')).title()
    while len(Campaign_Name.strip()) == 0:
        print('Warning!!!!')
        Campaign_Name = input("Campaign Name Required").title()
        continue
#------------ Start Date Time 
    while True:
        try:   
            StartDate = input('Enter Campaign Start Date as DDMMYYYY: ')
            StartDate1 = StartDate[0:2]
            StartDate2 = StartDate[2:4]
            StartDate3 = StartDate[4:8]
            StartDate = StartDate1+"-"+StartDate2+"-"+StartDate3
            StartTime = input('Enter Start Time(24-Hour) as HHMMSS: ')
            StartTime1 = StartTime[0:2]
            StartTime2 = StartTime[2:4]
            StartTime3 = StartTime[4:6]
            StartTime = StartTime1+":"+StartTime2+":"+StartTime3
            Start_Date = StartDate+" "+StartTime
            Start_Date = datetime.strptime(Start_Date,'%d-%m-%Y %H:%M:%S')
            
#-------------- End Date Time

            EndDate = input('Enter Campaign End Date as DDMMYYYY: ')
            EndDate1 = EndDate[0:2]
            EndDate2 = EndDate[2:4]
            EndDate3 = EndDate[4:8]
            EndDate = EndDate1+"-"+EndDate2+"-"+EndDate3
            EndTime = input('Enter End Time(24-Hour) as HHMMSS: ')
            EndTime1 = EndTime[0:2]
            EndTime2 = EndTime[2:4]
            EndTime3 = EndTime[4:6]
            EndTime = EndTime1+":"+EndTime2+":"+EndTime3
            End_Date = EndDate+" "+EndTime
            End_Date = datetime.strptime(End_Date,'%d-%m-%Y %H:%M:%S')
            
        except ValueError:
            print("Please enter Correct Date time format")
            #better try again... Return to the start of the loop
            continue
            time1 = End_Date
            time2 = Start_Date
            diff = time1-time2
            if Start_Date <= datetime.now():
                print("Voting Should not Start With past Hour")
                continue
            elif (diff.total_seconds()/3600) < 2:
                
                print('sorry')
                continue
            else:
                break
                
        except ValueError:
            print("Please enter Correct Date time format")
            continue
        else:
            break
    sql =  'insert into Campaign(Campaign_Name,Start_Date,End_Date) VALUES (%s,%s,%s)'
    val = (Campaign_Name,Start_Date,End_Date)
    cursor.execute(sql,val)
    conn.commit()
    print("~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~|")
    print("|~~~~~~~~~~~~~~",cursor.rowcount,")", "Campaign Created Sucessfully~~~~~~~~~~|")
    print("~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~|")
    return Candidate()
 
#---------------------Candidate
 
def Candidate(count= 0): 
    count += 1
    print('===========================================')
    print('|PLEASE ENTER DETAILS of CANDIDATE No.',count,'|')
    print('===========================================')
    Candidate_first_Name = (input('ENTER FIRST NAME :')).capitalize()
    while len(Candidate_first_Name.strip()) == 0:
        Candidate_first_Name = input("First Name Required")
        continue
    print()
    Candidate_last_Name = (input('Enter LAST NAME:')).capitalize()
    print()
    Party_name = (input('Enter Party_name:')).upper()
    while len(Party_name.strip()) == 0:
        Party_name = input("Party Name Required: ")
        continue
#----------Fatch Campaign_Id
        
    Sq = "SELECT * FROM campaign WHERE Campaign_Id=(SELECT max(Campaign_Id) FROM campaign)"
    cursor.execute(Sq)
    myresult = cursor.fetchone()
    Campaign_Id = int(myresult[0])

    sql =  'insert into Candidate(Candidate_first_Name,Candidate_last_Name,Party_name,Campaign_Id) VALUES (%s,%s,%s,%s)'
    val = (Candidate_first_Name,Candidate_last_Name,Party_name,Campaign_Id)
    cursor.execute(sql,val)
    conn.commit()
    print("=================================================")
    print("|  (",cursor.rowcount, ")  Record Added Sucessfully |")
    print("=================================================")
    
    if count == 1:
        return Candidate(count)
    else:
        print("==========================================================")
        print("|  Press 'A' to add New candidate n\ Press 'F' to Finish |")
        print("==========================================================")
        Dec2 = (input()).upper()
        if Dec2== 'A':
            return Candidate(count)
        else:
            print("=============================================================================")
            print("||  Congratulations!! you have finished adding", count, "Members for the Election  ||")
            print("=============================================================================")

#------------Publish Result----------------------
def Publish_Result():
    cursor=conn.cursor()
    sqlqry= "select Campaign_Name,Campaign_Id from campaign;"
    cursor.execute(sqlqry)
    vcmp = cursor.fetchall()
    listt = (tabulate(pd.DataFrame(vcmp).set_index([0]), headers=['Campaign Name','Campaign Id'],tablefmt= "psql"))
    print(listt)
    vcmp = pd.DataFrame(vcmp)
    Campdata = int(input("Select Campaign to view result"))
    customer_name = (vcmp.loc[vcmp[1] == Campdata, 0].iloc[0])
    cursor = conn.cursor()
    sql = f'SELECT votes.Campaign_Id, votes.Candidate_Id, votes.Voter_ID, VoteDateTime,concat(Candidate_first_Name," ",Candidate_last_Name)AS "Candidate Name" ,Party_name,Campaign_Name,End_Date FROM votes INNER JOIN candidate ON votes.Candidate_Id = candidate.candidate_Id INNER JOIN campaign ON votes.Campaign_Id = campaign.Campaign_Id WHERE  votes.Campaign_Id ="{Campdata}"'
    cursor.execute(sql)
    votesdata = pd.DataFrame(cursor.fetchall())
    dlist = votesdata.groupby([4,5])[2].count().sort_values(ascending = False)
    Newlist = dlist.reset_index()
    Newlist.rename(columns={4: 'Candidate Name',5:'Party',2: 'Vote Count'},inplace=True)
    #========= Table==============
    Newlist = Newlist.style \
        .set_caption(f'{customer_name} Voting Result') \
        .set_table_styles([
            {'selector': 'caption', 'props': [('font-size', '20px'), ('color', '#fc8d59')]},
            {'selector': 'thead th', 'props': [('background-color', '#3182bd'), ('color', 'black'), ('font-size', '15px')]},
            {'selector': 'tbody td', 'props': [('font-size', '16px')]}
        ]) \
        .set_properties(**{'text-align': 'center', 'border': '20px  #fcbb88'}) \
        .background_gradient(cmap='coolwarm')

    print("**Candidate having 0 votes wiill not appear in this list")
    display(Newlist)
    
#-------------------Voter 
def Voter_login(count = 0):
    cursor=conn.cursor()
    sqlqry= "SELECT * FROM voter"
    cursor.execute(sqlqry)
    log = cursor.fetchall()
    user = pd.DataFrame(log)   
    count +=1
    if count <= 3:
        a = input("enter Voter ID: ")
        b = getpass.getpass("enter password: ")
        for i in user.index:
            if user[0][i]== a and user[1][i]==b:
                print('login Sucessful')
                global Ylogid
                Ylogid = a
                return Vote_for()
        else:
            print('Wrong Id or Password',3-count,' attempt left..')
            return Voter_login(count)            
    else:
        print('Login count exceede, ID-LOCKED')

def Vote_for():
    cursor=conn.cursor()
    sqlqry= "SELECT Campaign_Name,Campaign_Id FROM campaign WHERE End_Date>= CURRENT_TIMESTAMP"
    cursor.execute(sqlqry)
    vcmp = cursor.fetchall()
    listt = (tabulate(pd.DataFrame(vcmp).set_index([0]), headers=['Campaign_Name','Campaign_Id'],tablefmt= "psql"))
    print(listt)
    global camp
    Selectcampaign = int(input("Select Campaign for which you want to vote: "))
    sqCheck = pd.DataFrame(cursor.fetchall())
    sqCheck= f"SELECT Voter_ID, Campaign_Id FROM votes WHERE Voter_ID = '{Ylogid}' AND Campaign_Id = '{Selectcampaign}';"
    cursor.execute(sqCheck)
    sqCheck = pd.DataFrame(cursor.fetchall())
    for j in sqCheck.index:
        if sqCheck[0][j]== Ylogid and sqCheck[1][j]== Selectcampaign:
            print("You Already Voted For This Campaign Try another Campaign")
            return Vote_for() 
        else:
            break    
    camp = Selectcampaign
    Cand = f"SELECT CONCAT(Candidate_first_Name,' ',Candidate_last_Name) as 'Candidate Name',Candidate_Id FROM candidate WHERE Campaign_Id = '{Selectcampaign}'"
    cursor.execute(Cand)
    candlist = pd.DataFrame(cursor.fetchall())


    #--------------------------------- working ---->   
    def button_pressed(event):
            global selected_button
            selected_button = event.widget['text']
            root.destroy()
    root = tk.Tk()
    root.title("Cast Your Vote")
    root.iconbitmap("Iconarchive-Blue-Election-Election-Vote.ico")
    root.geometry("300x450")
    root.configure(bg="SlateGray3")
    font = ("TkDefaultFont", 30)
    button_labels = candlist[0]
    buttons = []
    for label in button_labels:
        button = tk.Button(root, text=label, bg="light blue", width=20, height = 2)
        button.bind("<Button-1>", button_pressed)
        button.pack(pady= 10)
        buttons.append(button)
    root.mainloop() 
    # popup.mainloop()
    global Votecast
    Votecast = candlist[candlist[0]== selected_button]
    Votecast = int(Votecast[1])    

    #----------------Vote Excel Made-----------------------------
    cursor = conn.cursor()
    sql = 'INSERT INTO votes(Campaign_Id,Candidate_Id,Voter_ID,VoteDateTime) VALUES(%s,%s,%s,%s)'
    val = (camp,Votecast,Ylogid,datetime.now())
    cursor.execute(sql,val)
    conn.commit()

    print('|-----------------------------------------------------------------------------------------------------|')
    print('|        Congratulations.....!! Your Vote has been Registered Successfully thank you for Voting        |')
    print('|-----------------------------------------------------------------------------------------------------|')    

# End





