# Data-Git
Data Git GGC
#!/usr/bin/env python2
# -*- coding: utf-8 -*-

"""
Created on Mon Jun 25 10:22:58 2018

@author: vivek
@Module: Penn_Integration
"""
import os
import urllib2
import mysql.connector
import pandas as pd
import pprint
from xml.etree import cElementTree as ET

def check_names(path):
    if not os.path.exists(os.path.dirname(path)):
        os.makedirs(os.path.dirname)

log_in_url = 'lOGIN IN URL'
soapAction = "SOAP ACTION URL"

#headers = {'content-type': 'text/xml'}

http_headers = { 
 "User-Agent":"Mozilla/4.0 (compatible; MSIE 5.5;Windows NT)", 
 "Accept" :  "application/soap+xml,multipart/related,text/*", 
 "Cache-Control" :  "no-cache", 
 "Pragma" :  "no-cache", 
 "Content-Type" :  "text/xml; charset=utf-8;" ,
 "SOAPAction" : soapAction
 }



#DataBase Connection : Magentoid Pulling 
    


con = mysql.connector.connect(user='USER_NAME',
                              password='PASSWORD',
                              host='HOST',
                              database='DATBASE_NAME')

pd.read_sql("SELECT COLUMN_NAMES FROM TABLE_NAME;", con)
df = pd.read_sql("SELECT COLUMN_NAMES FROM TABLE_NAME;", con)
result_set = df[['vin','Msrp','modelyear','modelname']]


#List of Product id for loop

a=[130,96,91,213,102,97,90,101,89]


prettyp = pprint.PrettyPrinter(indent=4)


first_part = """
<x:Envelope xmlns:x="http://schemas.xmlsoap.org/soap/envelope/" xmlns:tem="http://tempuri.org/" xmlns:pen="http://schemas.datacontract.org/2004/07/PEN.Service.V3.ServiceTypes" xmlns:arr="http://schemas.microsoft.com/2003/10/Serialization/Arrays">
<x:Header>
<wsse:Security xmlns:wsse="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd" xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd">
<wsse:UsernameToken>
<wsse:Username>username</wsse:Username>
<wsse:Password Type="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-username-token-profile-1.0#PasswordText">password</wsse:Password>
</wsse:UsernameToken>
</wsse:Security>
</x:Header>
<x:Body>
<tem:GetRates>
<tem:request>
<pen:DealInfo>
<pen:APR>0.000010</pen:APR>
<pen:CarStatus>NEW</pen:CarStatus>
<pen:DealType>LEASE</pen:DealType>
<pen:DealerID>16856</pen:DealerID>
<pen:EffectiveDate>01/12/2017</pen:EffectiveDate>
<pen:FinanceTerms>
<arr:int>36</arr:int>
</pen:FinanceTerms>
<pen:FinancedAmount>19855</pen:FinancedAmount>
<pen:FirstOwner>true</pen:FirstOwner>
<pen:InServiceDate>01/12/2017</pen:InServiceDate>
<pen:Language>ENGLISH</pen:Language>
<pen:Mileage>10</pen:Mileage>
<pen:UserRole>FI_MANAGER</pen:UserRole>"""

second_part = """                    
<pen:VIN>%s</pen:VIN>
<pen:VehiclePurchasePrice>%s</pen:VehiclePurchasePrice> """

third_part  ="""                  
</pen:DealInfo>
<pen:Products>
<pen:DSIProductInfo>
<pen:EndingMileage>0</pen:EndingMileage>
<pen:IncludeWrapRates>false</pen:IncludeWrapRates>
<pen:LenderCode>?</pen:LenderCode>
<pen:ModelYear>2000</pen:ModelYear>
<pen:ProductID>{}</pen:ProductID>
<pen:RateBookCode>?</pen:RateBookCode>
<pen:StartingMileage>5</pen:StartingMileage>

<pen:VehicleID />
</pen:DSIProductInfo>
</pen:Products>
<pen:TestRequest>false</pen:TestRequest>
</tem:request>
</tem:GetRates>
</x:Body>
</x:Envelope>
"""

 
for index, row in df.iterrows():
    year=(row["modelyear"])
    model=(row["modelname"])
    login_xml = first_part + second_part %(str(row["vin"]),str(row["Msrp"]))
                
   
    
#Product id looping{val:Productid}
    for val in a:
        login_xml_final = login_xml + third_part.format(val)
    
        request_object = urllib2.Request(log_in_url,login_xml_final,http_headers)
        response = urllib2.urlopen(request_object)
        html_string = response.read()
        tree = ET.ElementTree(ET.fromstring(html_string))
        
        path = "LOCAL_PATH/{model}/{year}/{z}/{z}".format(model=model,year=year,z=val)
        os.makedirs(path)
        with open(path+".xml", "w") as f:
            f.write(html_string



           
