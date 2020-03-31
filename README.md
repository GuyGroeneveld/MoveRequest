This is a powershell module that contains functions I created as I was tired typing allways the same commands.

The source must be a live result form a "Get-MoveRequestStatistics -Includereport" or from an exported XML of it.

Copy the files to C:\Users\<user>\Documents\WindowsPowerShell\Modules\MoveRequest and you are done.                                       

Note that the Get-MoveRequestStatistics command may take time. If you have several move request to analyse, it is better to use the function Get-MoveRequestReport or Get-MoveRequestReportsFromBatch and then use the XMLs.

Get-MoveRequestFailures search the move request report for all failures that did occurs for the move request. This allows you to see the full failure history and take the correction action if needed. By default we exclude the permission mapping errors as they may flood your results, you can expressly request them if needed  with specify the parameter PermissionMappingError.   In case of Permission mapping, we retrieve the SID and mailbox folder involved.

Get-MoveRequestConnections search the move request report for all connections that did occurs for the move request. This allows you to see the full connection history, the MRS, MRS Proxy and Mailbox servers in use. The HLB parameter allows to retrieve the repartition of requests among the MRSProxy servers. This can help in determining if the load balancer is performing correctly. Proxyonly will get only the connection to on premises. Bear in mind that for Exchange 2013 and 2016, the CAS contacted will proxy the request. If the destination mailbox is 2013 or 2016 we end up to use the MRSProxy on the Mailbox server of the moving mailbox. If the mailbox is 2007 or 2010 we end up on a 2013/2016 Mailbox role which connect then to the remote mailbox server.

Get-MoveRequestThrottlingEvents search the move request report for all stalled events that did occurs for the move request. This allows you to determine when the move request got stalled and why. This can be useful when you want to determine why a mailbox move took longer than expected. The stalled entries will reflect by their name what throttled them. If a mailbox move request is constantly throttled and taking too much time to be moved, the easiest solution is to recreate the move request.

 

Get-MoveRequestSourceLatency search the move request report for source latency that did occurs for the move request. This allows you to determine when the move request got stalled and why. This can be useful when you want to determine why a mailbox move took longer than expected. Bear in mind that what matter is the average, the recommendation being to have it as low as possible. You can have peaks and be perfectly fine. Having the latency history helps determining if there's been a network issue and then look for throttling events or failures at the corresponding time.
Get-MoveRequestHistory search the move request report for historical events like move creation, incremantal sync, move request parameter change and fatal failures.
Get-MoveRequestThroughputFromReport attempt to retrieve the move request throughput from the report. The entries in the report are no showing up the bytes transfered but rather the replicated data from the mailbox. We analyze the report message and extract the data.
The report messages are looking like this one:

Copy progress: 1388/1598 messages, 546.1 MB (572,582,873 bytes)/621.9 MB (652,138,174 bytes), 68/79 folders completed.
On this entry we have replicated 546.1 MB from a source mailbox size of 621.9 MB
By keeping this value and getting the next one and comparing the replicated size and the minutes between the two report entries we determine the amount of data sent per minutes. If you compare the TotalMailBoxSize with BytesTransferred you will see that we transfer more bytes than the data. We don't have the values per message. Thus we get the percentage difference between TotalMailBoxSize and BytesTransferred. We then apply this percentage increase to all the measured values. Deltasize is the data related to mailbox size and Deltabytes is the adjusted one.
Here is an example output:
    StartTime            : 9/23/2017 12:05:34 AM
    EndTime              : 9/23/2017 12:20:20 AM
    Mailbox              : user20
    MigrationBatch       : FTC-Btach
    Already migrated     : 64,563,756
    Current Mailbox Size : 94,789,523
    deltasize            : 64,563,756
    deltabytes           : 73,675,389
    Percent Migrated     : 64.71%
    MegaBytesPerMinute   : 5
    BytesPerMinute       : 4,989,716
    MinutesSpent         : 15
    Message              : Copy progress: 1282/2021 messages, 61.57 MB (64,563,756 bytes)/90.4 MB (94,789,523 bytes), 17/31 folders completed.
  ----------------------------------------------------------------------------------------------------------------------------------------- 
  THE RESULT IS AN INDICATION. THE ONLY WAY TO HAVE REAL THROUGHPUT VALUES IS BY LOOKING AT THE LIVE THROUGHPUT OF A MAILBOX BEING MOVED
  -----------------------------------------------------------------------------------------------------------------------------------------
  My tests showed an average of more or less 20% difference between live and report.
  For live throughput you can look at the BytesTransferredPerMinute property of a move request currently moved as this attribute gets only documented
  when the mailbox is actively moved. You can also use the function from this module Get-MoveRequestLiveThroughputPerBatch.
Get-MoveRequestReport will get the move request staistics including the report and export it to an XML file per move request. We can automatically compress the result. This is usefull if you plan to move the data to another server or if you need to send the data to support. Be carefull about your disk space if you don't compress as the xml can become pretty big sometimes.

Get-MoveRequestReportsFromBatch will get the move request statistics including the report and export it to XML files.  We can automatically compress the result. This is usefull if you plan to move the data to another server or if you need to send the data to support. Be carefull about your disk space if you don't compress as the xml can become pretty big sometimes.

Get-MoveRequestLiveThroughputPerBatch get all the move request statistics for mailbox in the "InProgress" state and sum the BytesTransferredPerMinute.We then save the data per mailboxes plus the summarized data. In the chosen destination path we append the data everytime the function run if there are mailboxes currently transfering bytes.

CurrentPerMailboxBytesTransfered.csv will contain data per mailbox
CurrentTotalBytesTransfered.csv will contain summarized data and the effective amount of mailbox currently transferring data
The function returns the amount of mailboxes being at the "Inprogress" state. You can use this to determine if you will run the command again. For example you set a "Do While" powershell loop with a sleep of 5 minutes to get the status every five minutes. When the function returns 0 you are done. You can also use the scheduler, but this gets more complicated as you will need Office 365 creds to open the session.
Get-MoveRequestTiming get all the timestamps. If you want to know when the move was created, when it did start, when it did finish the initial seeding, when is the next incremental sync, when it did complete, you can use this function.
