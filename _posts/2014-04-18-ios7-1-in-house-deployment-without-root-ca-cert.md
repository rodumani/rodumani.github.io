---
title: iOS7.1 in-house deployment without root-ca cert
author: Rodumani
layout: post
permalink: /2014/04/ios7-1-in-house-deployment-without-root-ca-cert/
categories:
  - Development
  - iOS
---
iOS 7.1 blocked in-house deployment via http. It requires https with Root CA signed certificate. (Not impossible with self-signed cert but the certificate should be installed on each device.)

However, there is powerful Root CA Signed free file hosting service, Dropbox.

You can use in-house deployment via Dropbox download link because this link uses https protocol signed with Dropbox SSL Cert.

&nbsp;

Here are the steps.

1. In XCode, do Product &#8211; Archive.

2. Distribute as &#8220;Save for Enterprise or Ad Hoc Deployment&#8221; at Organizer.

3. Before save it, check &#8220;Save for Enterprise Distribution&#8221;. Fill Application URL with dummy URL (ex. http://example.com).

4. Upload exported .ipa and .plist to dropbox.

5. In dropbox, click .ipa file&#8217;s &#8220;Share link&#8221;.

6. You will see the dropbox file download page. Right-click the download button and &#8220;Copy link address&#8221;.

7. The copied url will looks like https://dl.dropboxusercontent.com/s/[randomHash]/example.ipa?dl=1&token_hash=[token hash] .  
Delete after &#8220;?&#8221;, and it should be form as https://dl.dropboxusercontent.com/s/[randomHash]/example.ipa

8. Open .plist file and replace http://example.com with https://dl.dropboxusercontent.com/s/[randomHash]/example.ipa

9. Get direct download link of .plist file same as getting .ipa file&#8217;s link (5-7).

10. Use &#8220;itms-services://?action=download-manifest&url=itms-services://?action=download-manifest&url=<span style="color: #ff0000;">https://dl.dropboxusercontent.com/s/[randomHash]/example.plist<span style="color: #000000;">&#8221; as link of in-house deployment. </span></span>

11. It works!