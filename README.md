# How-to-Call-Salesforce-API-from-Lightning-Component
![Header Image ](https://github.com/amitastreait/How-to-Call-Salesforce-API-from-Lightning-Component/blob/master/Call%20API%20From%20Lightning%20Component/HeaderImage.png)

You may have heard about that we can not make the Salesforce API Call directly from the lightning component. If you do make the callout from the lightning component then you will get the below error

# INVALID_SESSION_ID:This session is not valid for use with the API.

For Example when you will execute the below code from the Developer Console then you will get the Valid Response

```

HTTP h = new HTTP();
HTTPRequest req = new HTTPRequest();
HttpResponse resp = new HttpResponse();
 
req.setMethod('GET');
req.setHeader('Authorization', 'Bearer ' + UserInfo.getSessionId()); 
req.setEndpoint(URL.getSalesforceBaseUrl().toExternalForm() + '/services/data/v41.0/query?q=Select+Id,+Name+From+Account');
resp = h.send(req);
System.debug('#### Response Status '+resp.getStatus());
System.debug('#### Response Status Code '+resp.getStatusCOde());
System.debug(resp.getBody());

```


But if you will call the same block from the Lightning Component then you will get "INVALID_SESSION_ID:This session is not valid for use with the API." Error.

I gone through with this error and found THIS SALESFORCE DOCUMENT where they have specified why we can not make API Call from JavaScript Code.

After reading the document, I have come up with 2 Solutions that are given below: -

1 - Connected App and Named Credentials as Callout Endpoints and Auth Providers

    Connected App
    Auth Provider
    Named Credentials as Callout Endpoints
    For Complete Tutorial Visit This Link.

2 - Using VF Page - VF page will be used to get the Session Id of the current log in user and this VF page will be used into Apex class for fetching the Session Id.

In this tutorial we will use VF page for making callout because it is an easy and simple method method. Follow the below steps

Step1 - Open Developer Console, File -> New -> VisualForce Page -> Enter Name "GetSessionIdVF" -> OK. Use below code for VF Page

```

<apex:page >
 Start_Of_Session_Id{!$Api.Session_ID}End_Of_Session_Id
</apex:page>

```


Step2 - Create a New Class which will use this VF page to get the Session Id and making API Call Out. File -> New -> Apex Class -> Name it "ApiCallLightningComponent" -> OK. Use below code for the class.

```

public class ApiCallLightningComponent {
 /*
 * @Name : - fetchUserSessionId
 * @Description: - Call the VF page and get the Log In Use Session Id
 * @Params : - none
 * @ReturnType : - String
 */ 
 public static String fetchUserSessionId(){
 String sessionId = '';
 // Refer to the Page 
 PageReference reportPage = Page.GetSessionIdVF;
 // Get the content of the VF page
 String vfContent = reportPage.getContent().toString();
 System.debug('vfContent '+vfContent);
 // Find the position of Start_Of_Session_Id and End_Of_Session_Id 
 Integer startP = vfContent.indexOf('Start_Of_Session_Id') + 'Start_Of_Session_Id'.length(),
 endP = vfContent.indexOf('End_Of_Session_Id');
 // Get the Session Id
 sessionId = vfContent.substring(startP, endP);
 System.debug('sessionId '+sessionId);
 // Return Session Id
 return sessionId;
 }
 /*
 * @Name - makeAPICall
 * @Description - Responsible for making API Call out
 * @params - None
 * @ReturnType - String
 */ 
 @AuraEnabled 
 public static String makeAPICall(){
 String sessionId = fetchUserSessionId();
 HTTP h = new HTTP();
 HTTPRequest req = new HTTPRequest();
 HttpResponse resp = new HttpResponse();
 req.setMethod('GET');
 req.setHeader('Authorization', 'Bearer ' + sessionId); 
 req.setEndpoint(URL.getSalesforceBaseUrl().toExternalForm() + '/services/data/v41.0/query?q=Select+Id,+Name+From+Account');
 resp = h.send(req);
 System.debug('#### Response Status '+resp.getStatus());
 System.debug('#### Response Status Code '+resp.getStatusCOde());
 System.debug(resp.getBody());
 return JSON.serialize(resp.getBody());
 }
}

```

See the comments.

ApiCallLightningComponent class contains 2 methods fetchUserSessionId and makeAPICall, One for getting the Session Id and Other for making the Callout that will be called from the JavaScript.

Step3 - Create a lightning component, File -> New -> Lightning Component -> Name it "MakiAPICall" -> OK. Use below code for the Component

```

<aura:component implements="force:appHostable,flexipage:availableForAllPageTypes,
flexipage:availableForRecordHome,force:hasRecordId,
forceCommunity:availableForAllPageTypes,
force:lightningQuickAction"
controller="ApiCallLightningComponent"
access="global" >
<div class="slds-m-around_x-small">
<lightning:button variant="brand" label="Make CallOut"
iconName="action:apex" iconPosition="left" onclick="{!c.doHandleClick }" />
</div>
</aura:component>

```

Click on the Controller and use below code

```

({
doHandleClick : function(component, event, helper) {
helper.onHandleClick(component, event, helper);
}
})

```
Click on the helper and then use below code for helper

```

({
onHandleClick : function(component, event, helper) {
// Get the action of Controller (Apex) Class
var action = component.get('c.makeAPICall');

// set the callback which will return the response from apex
action.setCallback(this, function(response){
// get the state
var state = response.getState();
if( (state === 'SUCCESS' || state ==='DRAFT') && component.isValid()){
// get the response
var responseValue = response.getReturnValue();
// Parse the respose
var responseData = JSON.parse(responseValue);
alert(responseData);
//alert(responseData.totalSize);
console.log(responseData);
} else if( state === 'INCOMPLETE'){
console.log("User is offline, device doesn't support drafts.");
} else if( state === 'ERROR'){
console.log('Problem saving record, error: ' +
JSON.stringify(response.getError()));
} else{
console.log('Unknown problem, state: ' + state +
', error: ' + JSON.stringify(response.getError()));
}
});
// send the action to the server which will call the apex and will return the response
$A.enqueueAction(action);
}
})

```

Step4 - In Final Step Test the Component using Lightning Application. File -> New -> Lightning Application -> Enter Name -> Submit -> Use below Code

```

<aura:application extends='force:slds' >
<c:MakiAPICall/>
</aura:application>

```

Note: - If Your component name is different then use the name of your component instead of MakiAPICall.

OutPut

![Output Image ](https://github.com/amitastreait/How-to-Call-Salesforce-API-from-Lightning-Component/blob/master/Call%20API%20From%20Lightning%20Component/Call%20API%20From%20Lightning%20Component.gif)

