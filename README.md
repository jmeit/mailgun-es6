#mailgun-es6

The goal of this was to create an up-to-date library that interacts with the Mailgun API using only native JS modules.  It uses ES6 Classes and other features which requires it to be ran with the --harmony flag on node or with iojs.  When they merge this shouldn't be an issue. While all of the code is functional, I would not consider this stable until the test suite is complete. mailgun-es6 MIT licensed.

## Installation

```sh
$ npm install mailgun-es6
```

## Set Up
```js
var MailGun = require('mailgun-es6');
var mailGun = new MailGun({
  privateApi: 'Your Private API Key',
  publicApi: 'Your Public API Key',
  domainName: 'The domain on your account'
});
```

## Usage

### Messages

#### Sending Messages - [POST /\[domain\]/messages](https://documentation.mailgun.com/api-sending.html#sending)
```js
mailGun.sendEmail({
  to: 'to@email.com',
  from: 'from@email.com',
  subject: 'Email Subject',
  body: 'Email Text'
}[, domainName]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

The four required fields are above.  Any valid Mailgun options can be passed here in addition to these 4 fields. The Domain name passed in the function will supersede the default domain name if specified. The Mailgun descriptions are below along with the current limitations of this library.


| Parameter | Description |
| --- | --- |
| to | Email address of the recipient(s). Example: "Bob <bob@host.com>". You can use commas to separate multiple recipients. Although 'to' is the only required string, you can also submit 'cc' and 'bcc' |
| html  | An HTML version of the message you are sending |
| o:tag	| Tag string. See [Tagging](https://documentation.mailgun.com/user_manual.html#tagging) for more information. |
| o:campaign	| Id of the campaign the message belongs to. See [Campaign Analytics](https://documentation.mailgun.com/user_manual.html#um-campaign-analytics) for details. |
| o:deliverytime	| Desired time of delivery. See [Date Format](https://documentation.mailgun.com/api-intro.html#date-format). Note: Messages can be scheduled for a maximum of 3 days in the future. |
| o:dkim	| Enables/disabled DKIM signatures on per-message basis. Pass yes or no |
| o:testmode	 | Enables sending in test mode. Pass yes if needed. See [Sending in Test Mode](https://documentation.mailgun.com/user_manual.html#manual-testmode) |
| o:tracking | Toggles tracking on a per-message basis, see [Tracking Messages](https://documentation.mailgun.com/user_manual.html#tracking-messages) for details. Pass yes or no. |
| o:tracking-clicks	| Toggles clicks tracking on a per-message basis. Has higher priority than domain-level setting. Pass yes, no or htmlonly. |
| o:tracking-opens	| Toggles opens tracking on a per-message basis. Has higher priority than domain-level setting. Pass yes or no. |
| h:X-My-Header	| h: prefix followed by an arbitrary value allows to append a custom MIME header to the message (X-My-Header in this case). For example, h:Reply-To to specify Reply-To address. |
| v:my-var	| v: prefix followed by an arbitrary name allows to attach a custom JSON data to the message. See [Attaching Data to Messages](https://documentation.mailgun.com/user_manual.html#manual-customdata) for more information. |

#### Retrieving Stored Messages - [GET domains/[domain]/messages](https://documentation.mailgun.com/api-sending.html#retrieving-stored-messages)
```js
mailGun.getStoredMessages([msgId, domain]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

This function works with both retrieving the list of stored messages as well as single stored messages.  If no parameters as passed, it will return an array of objects are the stored messages.  Internally, this calls [Retrieving Events](#retrieving-events) with a event=stored filter. For more control on returned events, use the retrieving events call. If a msgId is passed, then it will look for that specific message.  Again, the domain parameter will overwrite the constructors domainName.

#### Deleting Stored Messages - [DELETE domains/[domain]/messages](https://documentation.mailgun.com/api-sending.html#deleting-stored-messages)
```js
mailGun.deleteStoredMessages([msgId, domain]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

This function deletes a stored message.  

### Domains
#### Retrieving Domain Information - [GET /domains/\[domain\]](https://documentation.mailgun.com/api-domains.html#domains)
```js
mailGun.getInformation([domain]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

This function will either retrieve all domain records for the associated account or a single domain if 'domain' is passed as a parameter.

#### Add New Domain - [POST /domains](https://documentation.mailgun.com/api-domains.html#domains)
```js
mailGun.addNewDomain('newDomain.com', 'smtpPassword', {
  spamAction: 'disable' || 'tag',
  wildCard: true || false
});

```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

This endpoint associates a new domain with a current Mailgun account. Spam action refers to either `disable` which will disable spam filtering and `tag` which will tag with a spam header. More information can be found about the [Spam Filter](https://documentation.mailgun.com/user_manual.html#um-spam-filter).  Wildcard enables the added account to also respond to subdomains. (e.g., kyle@sub.domain.com) It is recommended that all four fields are passed into the function, however the domain and password are the only required fields.  spamAction will default to `disable` and wildCard will default to `false`.

#### Delete Domain - [DELETE /domains](https://documentation.mailgun.com/api-domains.html#domains)
```js
mailGun.deleteDomain(domainToDelete);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

This method deletes a current domain associated with the account. `domainToDelete` is required and will not assume default domain as this is a destructive action.

#### Retrieving SMTP Credentials - [GET /domains/\[domain\]/credentials](https://documentation.mailgun.com/api-domains.html#domains)
```js
mailGun.getSmtpUsers([domain]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

This function will respond with the current SMTP credentials for the specified domain. If no domain is passed, it will return the domain set at object creation. This does not respond with the passwords. This only responds with a list of available logins.  

#### Add SMTP Credentials - [POST /domains\[<domain\]/credentials](https://documentation.mailgun.com/api-domains.html#domains)
```js
mailGun.addSmtpUser(username, password[, domain]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

This function adds a SMTP user to the account. SMTP users are not needed unless you're implementing a SMTP connection.

#### Update SMTP Credentials - [PUT /domains\[<domain\]/credentials/\[login\](https://documentation.mailgun.com/api-domains.html#domains)
```js
mailGun.updateSmtpUser(username, password[, domain]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

This function updates a current SMTP user for a domain.  This will not 'upsert' a user. Also, Mailgun has limited changing only the password.  To change the username, delete the target and create a new one.

#### Delete SMTP Credentials - [DELETE /domains\[<domain\]/credentials/\[login\]](https://documentation.mailgun.com/api-domains.html#domains)
```js
mailGun.deleteSmtpUser(username, password[, domain]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

This function removes a current SMTP user.


#### Get Connection Settings - [GET /domains/\]domain\]/connection](https://documentation.mailgun.com/api-domains.html#domains)
```js
mailGun.getConnectionSettings([domain]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

This function returns the connection settings for a domain.  The first setting (require_tls) is true if the domain is set to require a TLS connection to interact with the server. The second setting (skip_verfication) determines if the certificate and hostname verification should be skipped. See Update Connection Settings for full information.

#### Update Connection Settings - [PUT /domains/\[domain\]/connection](https://documentation.mailgun.com/api-domains.html#domains)
```js
mailGun.updateConnectionSettings(requireTLS, skipVerification[, domain]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

From the Mailgun Docs:

|Parameter | Description |
|----------|-------------|
| requireTLS | If set to True this requires the message only be sent over a TLS connection. If a TLS connection can not be established, Mailgun will not deliver the message.If set to False, Mailgun will still try and upgrade the connection, but if Mailgun can not, the message will be delivered over a plaintext SMTP connection. The default is False. |
| skipVerification | If set to True, the certificate and hostname will not be verified when trying to establish a TLS connection and Mailgun will accept any certificate during delivery. If set to False, Mailgun will verify the certificate and hostname. If either one can not be verified, a TLS connection will not be established. The default is False. |

### Stats

#### Get Domain Stats - [GET /\[domain\]/stats](https://documentation.mailgun.com/api-stats.html)
```js
mailGun.getStats([options, domain]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

This function takes an optional options argument that will take an object or a string and use it as a search query.  The query options are listed [here](https://documentation.mailgun.com/api-stats.html).  If a string is passed, it should be in the format of `var1=val1&var2=val2`.

#### Delete Domain Stats - [GET /\[domain\]/stats](https://documentation.mailgun.com/api-stats.html)
```js
mailGun.deleteStats(tagName[, domain]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

When a tag is applied to an email set, it is recorded by MailGun.  This function removes those custom counts. As far as I know, you can't remove the system ones like 'sent'. Though, then again, I never tried...

### Events
#### Retrieving Events - [GET /\[domain\]/events](https://documentation.mailgun.com/api-events.html#events)
```js
mailGun.getEvents([search, domain]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

This endpoint is somewhat confusing.  Please re-read all of this twice before sending angry emails or help make it less confusing.  Please first understand the [limits on logging for Mailgun domains](https://documentation.mailgun.com/api-events.html). For Free accounts you will only be able to see the last two days of logs.  For paid accounts, you will be able to see the last 30.  

With that in mind, let us explore the search option.  The same as above, you can either pass the search option as an object or as a string in 'var1=val1' format. I HIGHLY suggest that you use the object as there are some magical things that happen. (e.g., dates get converted to the [correct format](https://documentation.mailgun.com/api-intro.html#date-format)) With that in mind:

* Both 'begin' and 'end' accept either a text date (Jan 1, 2015) or the UNIX time in seconds.  Internally, the function uses new Date().  So any format that Date.parseDate() understands will work.  

* Date ranges only work if you specify (BOTH 'begin' and 'end') OR (either 'begin' or 'end' AND 'ascending')

In addition to 'begin', 'end', and 'ascending', any field from the [query options](https://documentation.mailgun.com/api-events.html#query-options) can be passed as key/value pairs in the search object.

Currently, this function does not support [event polling](https://documentation.mailgun.com/api-events.html#event-polling) as suggested by the Mailgun docs.

### Suppressions

#### Get Bounces - [GET /\[domain\]/bounces/\[address\]](https://documentation.mailgun.com/api-suppressions.html#bounces)
```js
mailGun.getBounces([limit, address, domain]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

When emails are bounced back from a server with a fatal error message, Mailgun does the good guy thing and black lists them for you automatically.  This endpoint will return a list the addresses that Mailgun as on your bounce list, along with the reason it is on the list (if it was assigned when the bounce record was created).

#### Add Bounces - [POST /\[domain\]/bounces](https://documentation.mailgun.com/api-suppressions.html#bounces)
```js
mailGun.addBounces({
    "address": "alice@example.com",
    "code": "550",
    "error": "Bounced",
    "created_at": "Thu, 13 Oct 2011 18:02:00 UTC"
  }[, domain]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

Mailgun allows you to batch create bounce lists or to create single records.  As far as required fields, the only key:value you need to have in the object is the address.  You can either pass a single object to this function or an array of objects up to 1000 at a time.

#### Delete Bounces - [DELETE /\[domain\]/bounces](https://documentation.mailgun.com/api-suppressions.html#bounces)
```js
mailGun.deleteBounces(address[, domain]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

This function removes email address from the bounce list.  Be careful when doing this. There may be a good reason why the email in on this list.

#### Get Unsubscribes - [GET /\[domain\]/unsubscribes/\[addres\]>](https://documentation.mailgun.com/api-suppressions.html#unsubscribes)
```js
mailGun.getUnsubscribes([limit, address, domain]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

This function returns the unsubscribes for the specified domain. If an address is given, it will return data about that single email.

#### Add Unsubscribes - [POST /\[domain\]/unsubscribes](https://documentation.mailgun.com/api-suppressions.html#add-a-single-unsubscribe)
```js
mailGun.addUnsubscribes({
    "address": "alice@example.com",
    "tag": "Tag to unsubscribe from.",
    "created_at": "Thu, 13 Oct 2011 18:02:00 UTC"
  }[, domain]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

When adding an email to be unsubscribed, you may specify a tag to unsubscribe from.  Also, and the default, '*' will unsubscribe from all mailings.

#### Delete Unsubscribes - [DELETE /\[domain\]/unsubscribes](https://documentation.mailgun.com/api-suppressions.html#delete-a-single-unsubscribe)
```js
mailGun.deleteUnsubscribes(address[, domain]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

This function removes an email from the domain unsubscribe list. Be careful, there may be a good reason why the email in on this list.

#### Get Complaints - [GET /\[domain\]/complaints/\[addres\]>](https://documentation.mailgun.com/api-suppressions.html#view-all-complaints)
```js
mailGun.getComplaints([limit, address, domain]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

This function returns the complaints for the specified domain. If an address is given, it will return data about that single email.

#### Add Complaints - [POST /\[domain\]/complaints](https://documentation.mailgun.com/api-suppressions.html#view-a-single-complaint)
```js
mailGun.addComplaints({
    "address": "alice@example.com",
    "created_at": "Thu, 13 Oct 2011 18:02:00 UTC"
  }[, domain]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

Manually adds a complaining address to the complaint list.

#### Delete Complaints - [DELETE /\[domain\]/complaints](https://documentation.mailgun.com/api-suppressions.html#add-multiple-complaints)
```js
mailGun.deleteComplaints(address[, domain]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

This function removes an email from the domain complaint list. Be careful, there may be a good reason why the email in on this list.

### Routes
#### Get Routes - [GET /routes/\[id\]>](https://documentation.mailgun.com/api-routes.html#actions)
```js
mailGun.getRoutes([id, limit, skip]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

This function returns either 1) A list of routes when limit, skip, both, or no arguments are passed or 2) one
record when id is given as they are unique.  

#### Add Routes - [POST /routes](https://documentation.mailgun.com/api-routes.html#actions)
```js
mailGun.addRoutes({
    "priority": 10,
    "description": "Any kind of helpful text you wish to remind yourself",
    "expression": "The [filter](https://documentation.mailgun.com/user_manual.html#routes) for the route",
    "action": "The [action](https://documentation.mailgun.com/user_manual.html#routes) for a true filter"
  });
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

This function adds a route record to the account. The filters that Mailgun should be first read about in the [official docs](https://documentation.mailgun.com/user_manual.html#routes)
before trying to be used.  They are extremely robust and offer many options.  Currently, this function only lets you associate one action with each route.


#### Update Routes - [PUT /routes](https://documentation.mailgun.com/api-routes.html#actions)
```js
mailGun.updateRoutes(routeId, {
    "priority": Number,
    "description": "Any kind of helpful text you wish to remind yourself",
    "expression": "The [filter](https://documentation.mailgun.com/user_manual.html#routes) for the route",
    "action": "The [action](https://documentation.mailgun.com/user_manual.html#routes) for a true filter"
  });
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

This function updates the route with the specified routeId. From the object in the second parameter, only one field is required and it does not matter
which field is passed.  If you do not want to change a field, just don't include it.  


#### Delete Routes - [DELETE /routes](https://documentation.mailgun.com/api-routes.html#actions)
```js
mailGun.deleteRoutes(routeId);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

This function removes a route from the domain list.

### Campaigns

#### Get Campaigns - [GET /\[domain\]/campaigns](https://documentation.mailgun.com/api-campaigns.html#campaigns)
```js
mailGun.getCampaigns([id, limit, skip, domain]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

While designing this function, I ran into a small problem: trying to use tests to determine if
id is passed or if it was just the limit.  To solve this, id must be passed as a string and
limit must be passed as a number.  If limit is passed as a string, no type conversion will happen,
and you're only going to get one result back.  That single result will be the search results for
the campaign id.

#### Update Campaigns - [PUT /\[domain\]/campaigns/\[id\]](https://documentation.mailgun.com/api-campaigns.html)
```js
updateCampaigns(id[, name, domain]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

Currently this function only has the ability to change the name of the campaign.  Stay tuned for the ability to
also change the id.


#### Delete Campaigns - [DELETE /\[domain\]/campaigns/\[id\]](https://documentation.mailgun.com/api-campaigns.html)
```js
mailGun.deleteCampaigns(idToDelete, domain);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

This function removes a campaign from the domain list. It also deletes all associated data for the campaign.

#### Get Campaign Events - [GET /\[domain\]/campaigns/\[id\]/\[eventType\]](https://documentation.mailgun.com/api-campaigns.html#events)
```js
mailGun.getCampaignsEvents(campId[, eventType, options, domain]);
```
##### Returns
```js
promise(MailGun_Response, Rejection_Message)
```

This function retrieves information about a certain campaign.  Make sure you read the [official docs](https://documentation.mailgun.com/api-campaigns.html#events)
about the ways you can sort this information. Any key can be passed into the options object.  (e.g., {recipient: 'bob@mail.com' } The accepted eventType's are:

* events - returns all of the following events
* stats - returns a summary of the campaign events
* clicks - returns data about what is clicked in the email
* opens - returns data about the number of emails that were actually opened
* unsubscribes - returns data about the number of unsubscribes
* complaints - returns data about the complaints the campaign has received
