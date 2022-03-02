---
layout: post
title: Simultaneously start multiple Sharepoint 2007 workflows using C# for a List
date: 2014-08-05 15:44
author: godwin3737
comments: true
categories: [C#, Multi threading, SharePoint, SharePoint 2007]
---
Its known that we can start SharePoint 2007 workflows <a title="programatically" href="http://www.tonytestasworld.com/post/howto-start-a-sharepoint-workflow-programmatically.aspx" target="_blank">programmatically</a>.  Here is the code that loops through all items in a list and starts the workflow for each item:
<pre>foreach(SPListItem item in list.Items)
{   
    SPListItem wrkItem =list.GetItemById(item.ID);   
    wrkflowmgr.StartWorkflow(wrkItem,wflassociation,
    wflassociation.AssociationData);
}
</pre>
However the SharePoint Paradox here is that you can start only one workflow at a time , and have to wait for it to take its sweet time to finish, before you start the workflow for next item in list.  So what do you do if you have ( like i had ) a requirement to start a workflow on multiple items in a list simultaneously ?  Obviously you post a question shouting for help in <a href="http://stackoverflow.com/questions/17465032/starting-sharepoint-2007-workflows-simutaneously-on-list-items-using-timer-job/23058993#23058993" target="_blank">stackoverflow</a>. I was told:
<blockquote><span style="font-size: 12pt;">there is no <strong>simultaneous</strong> method to start workflows for multiple list items at the same time.</span></blockquote>
But  i eventually figured out, that <span style="color: #008000;">Multi Threading is the solution.</span>

Steps
<ul>
	<li><span style="color: #0000ff;">Create a Class "startWorkflow" that starts the workflow for a list item</span></li>
</ul>
<pre>using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using Microsoft.SharePoint;
using Microsoft.SharePoint.Administration;
using Microsoft.SharePoint.Workflow;
using Microsoft.SharePoint.Utilities;
using System.Threading;

namespace SimultaneousWorkflows
{
	class startWorkflow
	{
		int itemID;
        string siteURl;
        Guid gid; // GUID of the Workflow

        public startWorkflow(int iID, string sURl, Guid guidWfl)
        {
            itemID = iID;
            siteURl = sURl;
            gid = guidWfl;
        }
        public void startworkflowThread()
        {
           SPSecurity.RunWithElevatedPrivileges(delegate()
            {
                 using (SPSite site = new SPSite(siteURl))
                    {
                        SPWeb web = site.RootWeb;
                        web.AllowUnsafeUpdates = true;
                        SPList list = web.Lists["Your List Name"];
                        SPWorkflowManager wrkflowmgr = site.WorkflowManager;
                        SPWorkflowAssociation wflassociation = list.WorkflowAssociations[gid];
                        SPListItem wrkItem = list.GetItemById(itemID);
                        wrkflowmgr.StartWorkflow(wrkItem, wflassociation, wflassociation.AssociationData);                                  
                    }   
            });                        
		}
	}
}</pre>
&nbsp;
<ul>
	<li><span style="color: #0000ff;">Implement Multithreading in the Timer Job Definition class file
</span></li>
</ul>
I am just adding the relevant code here . <strong>Note:</strong> i was creating a timer job that would start the workflow , hence i updated the Timer Job Definition file , you can however do the following changes to the relevant class in your project.
<p style="padding-left: 30px;"><span style="color: #ff9900;">Include the System.Collections.Generic and System.Threading namespaces.</span></p>

<pre> 
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using Microsoft.SharePoint;
using Microsoft.SharePoint.Administration;
using Microsoft.SharePoint.Workflow;
using Microsoft.SharePoint.Utilities;
using System.Threading;
using System.Diagnostics;
</pre>
<p style="padding-left: 30px;"><span style="color: #ff9900;">Create a variable holding the GUID value of Workflow you want to start.</span></p>

<pre> public Guid wfguid = "The GUID value of Workflow you want to start" ;</pre>
<p style="padding-left: 30px;"><span style="color: #ff9900;">Instantiate a Generic List ( Collection) of thread objects.</span></p>

<pre>List&lt;Thread&gt; resThreadCollection = new List&lt;Thread&gt;();</pre>
<p style="padding-left: 30px;"><span style="color: #ff9900;">Instantiate a Generic List ( Collection) of "startWorkflow" objects from the class you created above.</span></p>

<pre>List&lt;startWorkflow&gt; startWorkFlowCollection = new List&lt;startWorkflow&gt;();</pre>
<p style="padding-left: 30px;"><span style="color: #ff9900;">Update the Loop code as follows :</span></p>

<pre>foreach(SPListItem item in list.Items)
{     
    startWorkflow thisItem = new startWorkflow(item.ID, siteUrl, wfguid);
    startWorkFlowCollection.Add(thisItem);
}

// Create a thread for each item added into startWorkFlowCollection
if (startWorkFlowCollection.Count &gt; 0)
{
	foreach (startWorkflow itm in startWorkFlowCollection)
		{
			Thread MyThread = new Thread(new ThreadStart(itm.startworkflowThread));
			resThreadCollection.Add(MyThread);
		}
}

// Start the Threads	
 if (resThreadCollection.Count &gt; 0)
{
	foreach (Thread tobeStarted in resThreadCollection)
		{
			tobeStarted.Start();
                }
} 

</pre>
This is it , the workflows would start simultaneously now.
