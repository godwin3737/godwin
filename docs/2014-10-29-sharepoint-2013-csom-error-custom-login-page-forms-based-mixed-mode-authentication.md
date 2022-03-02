---
layout: post
title: SharePoint 2013 CSOM Error : Custom Login Page (Forms Based / Mixed mode authentication)
date: 2014-10-29 16:43
author: godwin3737
comments: true
categories: [SharePoint]
---
If you have a SharePoint On Premise installation with a Custom Login Page , you might encounter issues working with CSOM. The error would be primarily related to Authentication Failure. The following solution worked for me.

<span style="color: #0000ff;">Step 1</span>: The Custom Login Page should be placed in IIS Virtual Directory.

As a general practise the custom login page is normally placed in the 15 Hive Layouts folder. Move/Copy the ASPX file of your custom login page from the 15 hive layouts folder to  the following folder in IIS Virtual directory of the concerned Web Application:

"C:inetpubwwwrootwssVirtualDirectories&lt;yoursite&gt;_forms"

<span style="color: #0000ff;">Step 2</span>: Edit the Custom Login page file to make sure the master page url is correct.

After the movement of the ASPX file, open the file in a notepad and check for the <span style="font-weight: bold; color: #545454;">MasterPageFile </span><span style="color: #545454;">value and change it to the following </span><span style="color: #545454;">="~/</span><span style="font-weight: bold; color: #545454;">_layouts</span><span style="color: #545454;">/</span><span style="font-weight: bold; color: #545454;">15</span><span style="color: #545454;">/</span><span style="font-weight: bold; color: #545454;">simple</span><span style="color: #545454;">.</span><span style="font-weight: bold; color: #545454;">master</span><span style="color: #545454;">".</span>

<span style="color: #0000ff;">Step 3</span>: Change the Custom Login Page Url in Central Administration.

Navigate to Central Admin and change the Url defined in Custom Login Page for the Web App ( Manage Web Application --&gt; Select Web App --&gt; Authentication Providers) to the following :     "~/_forms/customlogin.aspx"

<span style="color: #0000ff;">Step 4</span>: For Mixed Mode Authentication Only.

If you are using Forms based authentication , you are already ready to go , however if you have mixed mode authentication then make sure to add the following event handler in the CSOM code :
<blockquote>
<p style="text-align: left; padding-left: 60px;"><span style="font-size: 10pt;">ClientContext clientContext = new ClientContext("<a href="http://servername/">http://servername/</a>");</span>
<span style="font-size: 10pt;"> clientContext.ExecutingWebRequest += new EventHandler&lt;WebRequestEventArgs&gt;(clientContext_ExecutingWebRequest);</span>
<span style="font-size: 10pt;"> Web site = clientContext.Web;</span>
<span style="font-size: 10pt;"> clientContext.Load(site);</span>
<span style="font-size: 10pt;"> clientContext.ExecuteQuery();</span></p>
<p style="text-align: left; padding-left: 60px;"><span style="font-size: 10pt;">static void clientContext_ExecutingWebRequest(object sender, WebRequestEventArgs e)</span>
<span style="font-size: 10pt;"> {</span>
<span style="font-size: 10pt;">                     <strong>e.WebRequestExecutor.WebRequest.Headers.Add(</strong><strong>"X-FORMS_BASED_AUTH_ACCEPTED"</strong><strong>, </strong><strong>"f"</strong><strong>);</strong></span>
<span style="font-size: 10pt;"> }</span></p>
</blockquote>
CSOM should work fine now !!

&nbsp;
