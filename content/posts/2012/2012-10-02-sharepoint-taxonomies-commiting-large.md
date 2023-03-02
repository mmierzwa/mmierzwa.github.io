---
blogger_id: tag:blogger.com,1999:blog-2283486810269355053.post-3603881166395201090
blogger_orig_url: http://byteloom.blogspot.com/2012/10/sharepoint-taxonomies-commiting-large.html
date: "2012-10-02T11:08:00Z"
tags:
- sharepoint 2010
title: SharePoint Taxonomies - Committing large amount of data to the MMD service
---

Once again about the SharePoint 2010 taxonomy service.  

As I wrote in my previous posts, loading data into MMD service automatically can be quite a challenge. First, you must [remember about illegal characters in terms labels]({{ site.baseurl }}{% post_url 2012/2012-09-20-sharepoint-taxonomies-labels-with %}). Second, you must [trace duplicates across sibling nodes]({{ site.baseurl }}{% post_url 2012/2012-09-28-sharepoint-taxonomies-struggling-with %}) in taxonomy trees. And this could not be the end of your problems especially if you plan to load some more data at one time.  
<!--more-->
Since MMD service is transactional you must first perform the [commit](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.taxonomy.termstore.commitall.aspx) operation in order to see your changes in MMD picker. From the programmatic point of view MMD service is a classic WCF service with http/https endpoints and [client proxy](http://msdn.microsoft.com/en-us/library/microsoft.sharepoint.taxonomy.taxonomysession.aspx) which is used in daily SharePoint development.  

When you try to perform some more complex set of operations (like adding or removing large number of terms and term sets) in one transaction you may come across the following error:  

```
An error occurred while receiving the HTTP response to
http://hostname:32843/3d1e97ecf7074067957b8efdac3ae1ab/MetadataWebService.svc.
This could be due to the service endpoint binding not using the HTTP protocol.
This could also be due to an HTTP request context being aborted by the server
(possibly due to the service shutting down). See server logs for more details.
```

Here is a full error description:  

```
System.ServiceModel.CommunicationException was caught  
  Message=An error occurred while receiving the HTTP response to http://hostname:32843/3d1e97ecf7074067957b8efdac3ae1ab/MetadataWebService.svc. This could be due to the service endpoint binding not using the HTTP protocol. This could also be due to an HTTP request context being aborted by the server (possibly due to the service shutting down). See server logs for more details.  
  Source=mscorlib  
  StackTrace:  
    Server stack trace:   
       at System.ServiceModel.Channels.HttpChannelUtilities.ProcessGetResponseWebException(WebException webException, HttpWebRequest request, HttpAbortReason abortReason)  
       at System.ServiceModel.Channels.HttpChannelFactory.HttpRequestChannel.HttpChannelRequest.WaitForReply(TimeSpan timeout)  
       at System.ServiceModel.Channels.RequestChannel.Request(Message message, TimeSpan timeout)  
       at System.ServiceModel.Channels.SecurityChannelFactory`1.SecurityRequestChannel.Request(Message message, TimeSpan timeout)  
       at System.ServiceModel.Dispatcher.RequestChannelBinder.Request(Message message, TimeSpan timeout)  
       at System.ServiceModel.Channels.ServiceChannel.Call(String action, Boolean oneway, ProxyOperationRuntime operation, Object[] ins, Object[] outs, TimeSpan timeout)  
       at System.ServiceModel.Channels.ServiceChannelProxy.InvokeService(IMethodCallMessage methodCall, ProxyOperationRuntime operation)  
       at System.ServiceModel.Channels.ServiceChannelProxy.Invoke(IMessage message)  
    Exception rethrown at [0]:   
       at System.Runtime.Remoting.Proxies.RealProxy.HandleReturnMessage(IMessage reqMsg, IMessage retMsg)  
       at System.Runtime.Remoting.Proxies.RealProxy.PrivateInvoke(MessageData& msgData, Int32 type)  
       at Microsoft.SharePoint.Taxonomy.Internal.IDataAccessReadWrite.Write(String data)  
       at Microsoft.SharePoint.Taxonomy.Internal.TaxonomyProxyAccess.<>c__DisplayClass40.<write>b__3f(IMetadataWebServiceApplication serviceApplication)  
       at Microsoft.SharePoint.Taxonomy.MetadataWebServiceApplicationProxy.<>c__DisplayClass2c.<runonchannel>b__2b()  
       at Microsoft.Office.Server.Security.SecurityContext.RunAsProcess(CodeToRunElevated secureCode)  
       at Microsoft.SharePoint.Taxonomy.MetadataWebServiceApplicationProxy.<>c__DisplayClass2c.<runonchannel>b__2a()  
       at Microsoft.Office.Server.Utilities.MonitoredScopeWrapper.RunWithMonitoredScope(Action code)  
       at Microsoft.SharePoint.Taxonomy.MetadataWebServiceApplicationProxy.RunOnChannel(CodeToRun codeToRun, Double operationTimeoutFactor)  
       at Microsoft.SharePoint.Taxonomy.MetadataWebServiceApplicationProxy.RunOnChannel(CodeToRun codeToRun)  
       at Microsoft.SharePoint.Taxonomy.Internal.TaxonomyProxyAccess.Write(String data)  
       at Microsoft.SharePoint.Taxonomy.Internal.DataAccessManager.Write(String data)  
       at Microsoft.SharePoint.Taxonomy.Internal.Sandbox.CommitSandbox()  
       at Microsoft.SharePoint.Taxonomy.TermStore.CommitAll()  
       at ...  

  InnerException: System.Net.WebException  
       Message=The underlying connection was closed: An unexpected error occurred on a receive.  
       Source=System  
       StackTrace:  
            at System.Net.HttpWebRequest.GetResponse()  
            at System.ServiceModel.Channels.HttpChannelFactory.HttpRequestChannel.HttpChannelRequest.WaitForReply(TimeSpan timeout)  
       InnerException: System.IO.IOException  
            Message=Unable to read data from the transport connection: An existing connection was forcibly closed by the remote host.  
            Source=System  
            StackTrace:  
                 at System.Net.Sockets.NetworkStream.Read(Byte[] buffer, Int32 offset, Int32 size)  
                 at System.Net.PooledStream.Read(Byte[] buffer, Int32 offset, Int32 size)  
                 at System.Net.Connection.SyncRead(HttpWebRequest request, Boolean userRetrievedStream, Boolean probeRead)  
            InnerException: System.Net.Sockets.SocketException  
                 Message=An existing connection was forcibly closed by the remote host  
                 Source=System  
                 ErrorCode=10054  
                 NativeErrorCode=10054  
                 StackTrace:  
                      at System.Net.Sockets.NetworkStream.Read(Byte[] buffer, Int32 offset, Int32 size)  
                 InnerException:   
```

The bottom exception of the stack trace - `System.Net.WebException` (_The underlying connection was closed: An unexpected error occurred on a receive_) - suggests that problem related with web application/IIS configuration rather than MMS service itself. [Andreas Cieslik](http://code2life.blogspot.com) already found the solution for this and [posted on his blog](http://code2life.blogspot.com/2011/10/recently-i-tried-to-publish-custom.html). In short - you must increase the default request restrictions in services `web.config` file.  

When changed this the `System.ServiceModel.CommunicationException` disappeared but I've got another error:

```  
System.TimeoutException was caught  
Message=The request channel timed out while waiting for a reply after 00:00:29.9570296.
Increase the timeout value passed to the call to Request or increase the SendTimeout
value on the Binding. The time allotted to this operation may have been a portion of a
longer timeout.
```

and the full error description:  

```
System.TimeoutException was caught  
  Message=The request channel timed out while waiting for a reply after 00:00:29.9570296\. Increase the timeout value passed to the call to Request or increase the SendTimeout value on the Binding. The time allotted to this operation may have been a portion of a longer timeout.  
  Source=mscorlib  
  StackTrace:  
    Server stack trace:   
       at System.ServiceModel.Channels.RequestChannel.Request(Message message, TimeSpan timeout)  
       at System.ServiceModel.Channels.SecurityChannelFactory`1.SecurityRequestChannel.Request(Message message, TimeSpan timeout)  
       at System.ServiceModel.Dispatcher.RequestChannelBinder.Request(Message message, TimeSpan timeout)  
       at System.ServiceModel.Channels.ServiceChannel.Call(String action, Boolean oneway, ProxyOperationRuntime operation, Object[] ins, Object[] outs, TimeSpan timeout)  
       at System.ServiceModel.Channels.ServiceChannelProxy.InvokeService(IMethodCallMessage methodCall, ProxyOperationRuntime operation)  
       at System.ServiceModel.Channels.ServiceChannelProxy.Invoke(IMessage message)  
    Exception rethrown at [0]:   
       at System.Runtime.Remoting.Proxies.RealProxy.HandleReturnMessage(IMessage reqMsg, IMessage retMsg)  
       at System.Runtime.Remoting.Proxies.RealProxy.PrivateInvoke(MessageData& msgData, Int32 type)  
       at Microsoft.SharePoint.Taxonomy.Internal.IDataAccessReadWrite.Write(String data)  
       at Microsoft.SharePoint.Taxonomy.Internal.TaxonomyProxyAccess.<>c__DisplayClass40.<write>b__3f(IMetadataWebServiceApplication serviceApplication)  
       at Microsoft.SharePoint.Taxonomy.MetadataWebServiceApplicationProxy.<>c__DisplayClass2c.<runonchannel>b__2b()  
       at Microsoft.Office.Server.Security.SecurityContext.RunAsProcess(CodeToRunElevated secureCode)  
       at Microsoft.SharePoint.Taxonomy.MetadataWebServiceApplicationProxy.<>c__DisplayClass2c.<runonchannel>b__2a()  
       at Microsoft.Office.Server.Utilities.MonitoredScopeWrapper.RunWithMonitoredScope(Action code)  
       at Microsoft.SharePoint.Taxonomy.MetadataWebServiceApplicationProxy.RunOnChannel(CodeToRun codeToRun, Double operationTimeoutFactor)  
       at Microsoft.SharePoint.Taxonomy.MetadataWebServiceApplicationProxy.RunOnChannel(CodeToRun codeToRun)  
       at Microsoft.SharePoint.Taxonomy.Internal.TaxonomyProxyAccess.Write(String data)  
       at Microsoft.SharePoint.Taxonomy.Internal.DataAccessManager.Write(String data)  
       at Microsoft.SharePoint.Taxonomy.Internal.Sandbox.CommitSandbox()  
       at Microsoft.SharePoint.Taxonomy.TermStore.CommitAll()  
       at ABB.Library.Publishing.SharePoint.CategoryImport.ClassificationProvider.CategoryTermStoreAdapter.Commit() in C:\work\Dev\07SharePoint\01Impl\ABB.Library.Publishing.SharePoint\ABB.Library.Publishing.SharePoint.CategoryImport\ClassificationProvider\CategoryTermStoreAdapter.cs:line 152  
       at ABB.Library.Publishing.SharePoint.CategoryImport.ImportProcess.Import() in C:\work\Dev\07SharePoint\01Impl\ABB.Library.Publishing.SharePoint\ABB.Library.Publishing.SharePoint.CategoryImport\ImportProcess.cs:line 49  
  InnerException: System.TimeoutException  
       Message=The HTTP request to 'http://hostname:32843/3d1e97ecf7074067957b8efdac3ae1ab/MetadataWebService.svc' has exceeded the allotted timeout of 00:00:30\. The time allotted to this operation may have been a portion of a longer timeout.  
       Source=System.ServiceModel  
       StackTrace:  
            at System.ServiceModel.Channels.HttpChannelUtilities.ProcessGetResponseWebException(WebException webException, HttpWebRequest request, HttpAbortReason abortReason)  
            at System.ServiceModel.Channels.HttpChannelFactory.HttpRequestChannel.HttpChannelRequest.WaitForReply(TimeSpan timeout)  
            at System.ServiceModel.Channels.RequestChannel.Request(Message message, TimeSpan timeout)  
       InnerException: System.Net.WebException  
            Message=The operation has timed out  
            Source=System  
            StackTrace:  
                 at System.Net.HttpWebRequest.GetResponse()  
                 at System.ServiceModel.Channels.HttpChannelFactory.HttpRequestChannel.HttpChannelRequest.WaitForReply(TimeSpan timeout)  
            InnerException:
            ...
```

`sendTimeout` is a strictly WCF thing, but unfortunately SharePoint taxonomy server side API does not provide any method for passing client side timeouts. You can do this only through `%SharePointRoot%\\WebClients\\Metadata\\client.config` file:  

```xml
<?xml version="1.0" encoding="utf-8" ?>  
<configuration>  
  <system.serviceModel>  
    <client>  
      <endpoint  
        name="http"  
        contract="Microsoft.SharePoint.Taxonomy.IMetadataWebServiceApplication"  
        binding="customBinding"  
        bindingConfiguration="MetadataWebServiceHttpBinding" />  
      <endpoint  
        name="https"  
        contract="Microsoft.SharePoint.Taxonomy.IMetadataWebServiceApplication"  
        binding="customBinding"  
        bindingConfiguration="MetadataWebServiceHttpsBinding" />  
    </client>  
    <bindings>  
      <customBinding>  
        <binding  
          name="MetadataWebServiceHttpBinding"  
                 receiveTimeout="00:00:30"  
                 sendTimeout="00:00:30"  
                 openTimeout="00:00:30"  
                 closeTimeout="00:00:30">  
          <security  
            authenticationMode="IssuedTokenOverTransport"  
            allowInsecureTransport="true" />  
          <binaryMessageEncoding>  
            <readerQuotas  
               maxStringContentLength="2147483647"  
               maxArrayLength="2147483647"  
               maxBytesPerRead="2147483647" />  
          </binaryMessageEncoding>  
          <httpTransport  
            transferMode="StreamedResponse"  
            maxReceivedMessageSize="2147483647"  
            authenticationScheme="Anonymous"  
            useDefaultWebProxy="false" />  
        </binding>  
        <binding  
          name="MetadataWebServiceHttpsBinding"  
                 receiveTimeout="00:00:30"  
                 sendTimeout="00:00:30"  
                 openTimeout="00:00:30"  
                 closeTimeout="00:00:30">  
          <security  
            authenticationMode="IssuedTokenOverTransport" />  
          <binaryMessageEncoding>  
            <readerQuotas  
              maxStringContentLength="2147483647"  
              maxArrayLength="2147483647"  
              maxBytesPerRead="2147483647" />  
          </binaryMessageEncoding>  
          <httpsTransport  
            transferMode="StreamedResponse"  
            maxReceivedMessageSize="2147483647"  
            authenticationScheme="Anonymous"  
            useDefaultWebProxy="false" />  
        </binding>  
      </customBinding>  
    </bindings>  
  </system.serviceModel>  
  <system.net>  
    <connectionManagement>  
      <add address="*" maxconnection="10000" />  
    </connectionManagement>  
  </system.net>  
</configuration>  
```

The values that must be increased are `sendTimeout` for proper bindings (line 21 for http and 42 for https). In my case it must have been changed to 10 minutes.  

Please note that both settings - maximum request length for all SP web services and WCF settings for MMD service - are server wide and will affect the whole machine (or farm in load balanced environment, since you will probably have to provide the same configuration on all web service machines). This may not be an acceptable solution on many production configurations especially in public networks (due to possible DoS threat) - I'm not sure how this affects other client APIs (like JS). This is rather a workaround.  

Happy SharePointing!

{{< blogspot_ref >}}