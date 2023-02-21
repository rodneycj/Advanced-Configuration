# Advanced Configuration
Use props.conf and transforms.conf to anonymize data. Audit Splunk environment through searches.

Abstract: This lab has 2 parts.  The first part is to use Splunk to transform data that has credit card numbers in it into anonymized data.  We can accomplish this via the props.conf and transforms.conf configuration files. The second part is analyzing.

Procedure – Detailed Lab Steps
----------------------------------------------------------------------------------------------------------------------------------------------------------------------

Take a snapshot.

SSH into the Splunk VM.

Since we have recreated our Splunk VM we need to do some housekeeping in order to complete this lab. Change the hostname of your Splunk VM to “siem” using your favorite text editor

     sudo nano /etc/hostname

Change the hostname of your VM from “ubuntu-20-04-server-template” to "siem" and save the file

Next, edit the hosts file to the new hostname.  

     sudo nano /etc/hosts

Replace “ubuntu-20-04-server-template” with "siem" and save the file

Reboot the VM 

     reboot 

When the VM comes back up make sure that the prompt is siem@siem:~$ after the VM starts

Take another snapshot

Boot into your VM and login via SSH

Using a text editor, create a log file named "creditcards.log" in the /var/log directory

     sudo nano /var/log/creditcards.log

Paste the following text into the log file

     transaction_id=1234aaff address=1234_credit_card_way credit_card_type=MasterCard account_number=5105105105105100 
     transaction_id=1234ab00 address=1235_credit_card_way credit_card_type=Visa account_number=4012888888881988
     
Make sure that there are only two lines and save the file as creditcards.log.

----------------------------------------------------------------------------------------------------------------------------------------------------------------------

Now we need to set Splunk to monitor the /var/log directory.  (if you have already done this you may need to change your data input settings). 

Go to Settings > Add Data > Monitor > Select Files & Directories > Then Browse to /var/log and select it.  

Click Next at the top

Select "constant value"

Host field value: "siem"   

Index: “main”

Check to make sure Splunk indexed the data by logging into Splunk and searching for the data.  Search for: 

    source="/var/log/creditcards.log"
    
You’ll notice that you can see that there is a field called account_number.  If this were a real company, we would be violating PCI-DSS compliance by storing the credit card number.  We need to anonymize this data.  

----------------------------------------------------------------------------------------------------------------------------------------------------------------------

Go back to the terminal and change the directory to /opt/splunk/etc/system/local   

There is no props.conf nor transforms.conf in that directory, so we need to add them.  Use your favorite editor to create props.conf and type in the file:

     sudo nano props.conf
     
Enter the following into the file:

     [host::siem]
     TRANSFORMS-anonymize_card = cc-anonymizer
     
Save the file.  You need to change the owner and group of the file to root
     
     sudo chown root:root ./props.conf
     
This will tell the Splunk instance to apply a transformation to data on the host: siem.

Next, create a new file in the same directory called transforms.conf and add the following

     [cc-anonymizer]
     REGEX = (?m)^(.*)account_number=(\d{5}\d{7})
     FORMAT = $1account_number=#########$
     DEST_KEY = _raw
     
Save the file.  Next, you need to change the owner and group of the file to root

     sudo chown root:root ./transforms.conf
     
Restart Splunk 

     sudo /opt/splunk/bin/splunk restart
     
----------------------------------------------------------------------------------------------------------------------------------------------------------------------
     
Create another credit card log

     sudo nano /var/log/creditcards2.log
     
Paste the following text into the log file and save:     

     transaction_id=1234aaff address=1234_credit_card_way credit_card_type=MasterCard account_number=5105105105105100 
     transaction_id=1234ab00 address=1235_credit_card_way credit_card_type=Visa account_number=4012888888881988
     
Log into Splunk and perform another quick search.  The data should be anonymized this time.

![091722_SIEM300-6_Lab Advanced configuration ](https://user-images.githubusercontent.com/123989567/220253019-37c24946-ea26-42f6-96ea-cb8dceb771c6.jpg)




