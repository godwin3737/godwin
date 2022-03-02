---
layout: post
title: SharePoint MySite - PowerShell script to get the "Activities I am following" preferences
date: 2012-03-28 13:15
author: godwin3737
comments: true
categories: [SharePoint]
---
So i was asked to collect the preferences of "Activities I am following" (image below) for all the users of our SP2010 My Site.
<p style="text-align: center;"><a href="http://gptp.azurewebsites.net/wp-content/uploads/2012/03/activities-i-am-following.jpg"><img class="size-full wp-image-69 aligncenter" style="border: 2px solid black;" title="activities-i-am-following" src="http://gptp.azurewebsites.net/wp-content/uploads/2012/03/activities-i-am-following.jpg" alt="" width="388" height="618" /></a></p>
The resources on web for this being  surprisingly limited , i was left with no other option but to turn to my cup of coffee  for solution . The coffees helped, the following script lists out the status of options available under "Activities i am following " of My Site Edit Profile page in SharePoint 2010 , for each user.
<pre><span style="color: #339966;">$mySiteURL = "https://xxxx"</span>
<span style="color: #339966;">$site = Get-SPSite $mySiteURL</span>
<span style="color: #339966;">$context = Get-SPServiceContext $site;</span>
<span style="color: #339966;">$upm = New-Object Microsoft.Office.Server.UserProfiles.UserProfileManager($context);</span>

<span style="color: #339966;">$val = "Username,ActivityName,Value"</span>
<span style="color: #339966;">Add-content -path "d:activity_preference.txt" -value $val</span>

<span style="color: #339966;">foreach ($profileupm in $upm.GetEnumerator())</span>
<span style="color: #339966;">{</span>
<span style="color: #339966;">$myprofile = $profileupm;</span>
<span style="color: #339966;">$usrprofile = $profileupm;</span>

<span style="color: #339966;">$am = New-Object Microsoft.Office.Server.ActivityFeed.ActivityManager($myprofile,
 $context)</span>
<span style="color: #339966;">$type = $am.GetType()</span>

<span style="color: #339966;">$methodInfo = $type.GetMethod("CopyBasicUserInfo", 
 [reflection.bindingflags]"nonpublic,instance",
 $null, $usrprofile.GetType(), $null)</span>

<span style="color: #339966;">$methodInfo.Invoke($am, $usrprofile)</span>

<span style="color: #339966;">$preftype = $am.ActivityPreferences.GetActivityPreferencesPerType();</span>

<span style="color: #339966;">foreach($pr in $preftype)</span>
<span style="color: #339966;">{</span>
<span style="color: #339966;">foreach($a in $am.ActivityTypes)</span>
<span style="color: #339966;">{</span>
<span style="color: #339966;">if ($pr.ActivityType -eq $a)</span>
<span style="color: #339966;">{</span>
<span style="color: #339966;">$val = $usrprofile["AccountName"].ToString() +
 "," + $a.ActivityTypeName.ToString() + 
 "," + $pr.IsSet.ToString()</span>

 <span style="color: #339966;">Add-content -path "d:activity_preference.txt" -value $val</span>
<span style="color: #339966;">}</span><span style="color: #339966;"> } </span><span style="color: #339966;"> }</span><span style="color: #339966;"> }</span></pre>
