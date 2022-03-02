---
layout: post
title: FTP C# Error : "The remote server returned an error: (530) Not logged in ."
date: 2013-06-25 08:16
author: godwin3737
comments: true
categories: [.Net, C#, FTP, SharePoint, SharePoint]
---
<h3>Recently working on a FTP solution using C# , i encountered an error <span style="color: #ff0000;">"</span><span style="color: #ff0000;">The </span><span style="color: #ff0000;">remote server returned an error: (530) Not logged in."</span></h3>
The code i used was following
<pre><span style="color: #ff0000;"><em>FtpWebRequest request = (FtpWebRequest)WebRequest.Create(<a href="ftp://xxxxxx/file.txt">ftp://xxxxxx/file.txt</a>); 
request.Method = WebRequestMethods.Ftp.UploadFile
request.Credentials = new NetworkCredential(usernameVariable, passwordVariable); </em></span></pre>
<em>What was more bewildering was if i modified the code to following, the solution was working fine. But this for obvious reasons is not an option as the username cannot be hardcoded</em>
<pre><span style="color: #0000ff;">//works but implausible to use in realtime solutions
request.Credentials = new NetworkCredential("dmn/#gsgs", password);  </span></pre>
Some googling revealed that special charcters create issues in the NetworkCredential Object. Hence some playing around worked for me, and it works irrespective of wether i do a FTPWebRequest or WebRequest.

<span style="text-decoration: underline;"><strong>Solution:</strong></span>

Instantiate NetworkCredential object with three paramters (username, password, domain) and make sure to normalize the string variables <span style="text-decoration: underline;">while instantiating</span> the NetworkCredential Object.

i.e:
<pre><span style="color: #00ff00;"><strong>//this works
request.Credentials = new NetworkCredential(usernameVariable.Normalize(),passwordVariable.Normalize(),domainVariable.Normalize());</strong> </span></pre>
Note: Normalize the string while instantiating the NetworkCredential object like above . The following doesnt work:
<pre><span style="color: #ff0000;">string usr = usernameVariable.Normalize()
string pwd= passwordVariable.Normalize()
string dom = domainVariable.Normalize()
request.Credentials = new NetworkCredential(usr,pwd,dom);  // wont work</span></pre>
<span style="color: #000000;"> </span>

<span style="color: #000000;">Hope it helps !!</span>
