# Rich-Notification-fcm-and-apns-

Note:- Uploaded code file representing iOS Push Notification Only. 
Note:- Add App transport Security>Allow Arbitrary = true/Yes in main app info.plist

Follow neccesssary steps to register device to recevie push notification as we do normally.

Here we have two cases one receving notification from apns i.e.., default Apple iOS Push Notification second from FCM (Firebase Cloud Messaging)

iOS Apns json :-
{
   "aps": {
       "alert": "Hello!",
       "sound": "default",
       "mutable-content": 1,
       "badge": 1
   },
   "data": {
       "attachment-url": "https://imagejournal.org/wp-content/uploads/bb-plugin/cache/23466317216_b99485ba14_o-panorama.jpg" //image url
   }
}

FCM JSON:-

	"to": "fNl_b5LHNtw:APA91bGs8wrkElxZaBNuAOc_qr5M5h0ATGx_SxvFEMI1KQ0t73v_INqcfBbkpNrPprsjeRwhfNMcsqgjkegKkDrL2IVUGI9LTj4EkwB58ebUySIEO1CzrskOlvM1PEx9h56Kfjvf_llt" //token,

 "mutable_content":true,
  "notification": {
    "title": "Its about time",
     "body": "To go online Amigo",
     "click_action": "NotificationCategoryIdentifier ForYourNotificationActions"
  },
  "data":{
   
    "attachment-url":"https://imagejournal.org/wp-content/uploads/bb-plugin/cache/23466317216_b99485ba14_o-panorama.jpg"// image url,
    
  }
}


You have to add Notification Extension in your project, File>New>Target>Notification Service Extension.
this service extension has its own bundle id and all, if in case don't work then create that bundle id in your apple developer account without creating new certificate for that.

this service extension will add two files one is its own info.plist and other is swift file.

open that swift file and replace that with following

iOS Apns in Case:-
//
//  NotificationService.swift
//  notify
//
//  Created by Nitin Bhatia on 08/01/18.
//  Copyright © 2018 Nitin Bhatia. All rights reserved.
//

import UserNotifications

class NotificationService: UNNotificationServiceExtension {

    var contentHandler: ((UNNotificationContent) -> Void)?
    var bestAttemptContent: UNMutableNotificationContent?

    override func didReceive(_ request: UNNotificationRequest, withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void) {
        self.contentHandler = contentHandler
        bestAttemptContent = (request.content.mutableCopy() as? UNMutableNotificationContent)
        
        func failEarly() {
            contentHandler(request.content)
        }
        
        guard let content = (request.content.mutableCopy() as? UNMutableNotificationContent) else {
            return failEarly()
        }
        
        guard let apnsData = content.userInfo["data"] as? [String: Any] else {
            return failEarly()
        }
        
        guard let attachmentURL = apnsData["attachment-url"] as? String else {
            return failEarly()
        }
        
        guard let imageData = NSData(contentsOf:NSURL(string: attachmentURL)! as URL) else { return failEarly() }
        guard let attachment = UNNotificationAttachment.create(imageFileIdentifier: "image.gif", data: imageData, options: nil) else { return failEarly() }
        
        content.attachments = [attachment]
        contentHandler(content.copy() as! UNNotificationContent)
    }
    
    override func serviceExtensionTimeWillExpire() {
        // Called just before the extension will be terminated by the system.
        // Use this as an opportunity to deliver your "best attempt" at modified content, otherwise the original push payload will be used.
        if let contentHandler = contentHandler, let bestAttemptContent =  bestAttemptContent {
            contentHandler(bestAttemptContent)
        }
    }

}

extension UNNotificationAttachment {
    static func create(imageFileIdentifier: String, data: NSData, options: [NSObject : AnyObject]?) -> UNNotificationAttachment? {
        
        let fileManager = FileManager.default
        let tmpSubFolderName = ProcessInfo.processInfo.globallyUniqueString
        let fileURLPath      = NSURL(fileURLWithPath: NSTemporaryDirectory())
        let tmpSubFolderURL  = fileURLPath.appendingPathComponent(tmpSubFolderName, isDirectory: true)
        
        do {
            try fileManager.createDirectory(at: tmpSubFolderURL!, withIntermediateDirectories: true, attributes: nil)
            let fileURL = tmpSubFolderURL?.appendingPathComponent(imageFileIdentifier)
            try data.write(to: fileURL!, options: [])
            let imageAttachment = try UNNotificationAttachment.init(identifier: imageFileIdentifier, url: fileURL!, options: options)
            return imageAttachment
        } catch let error {
            print("error \(error)")
        }
        
        return nil
    }
}

FCM in case:-
//
//  NotificationService.swift
//  notify
//
//  Created by Nitin Bhatia on 08/01/18.
//  Copyright © 2018 Nitin Bhatia. All rights reserved.
//

import UserNotifications

class NotificationService: UNNotificationServiceExtension {
    
    var contentHandler: ((UNNotificationContent) -> Void)?
    var bestAttemptContent: UNMutableNotificationContent?
    
    override func didReceive(_ request: UNNotificationRequest, withContentHandler contentHandler: @escaping (UNNotificationContent) -> Void) {
        self.contentHandler = contentHandler
        bestAttemptContent = (request.content.mutableCopy() as? UNMutableNotificationContent)
        
        func failEarly() {
            contentHandler(request.content)
        }
        
        guard let content = (request.content.mutableCopy() as? UNMutableNotificationContent) else {
            return failEarly()
        }
        
        guard let apnsData = content.userInfo["attachment-url"] as? String else {
            return failEarly()
        }
        //apnsData
        guard let attachmentURL = apnsData as? String else {
            return failEarly()
        }
        
        guard let imageData = NSData(contentsOf:NSURL(string: attachmentURL)! as URL) else { return failEarly() }
        guard let attachment = UNNotificationAttachment.create(imageFileIdentifier: "image.gif", data: imageData, options: nil) else { return failEarly() }
        
        content.attachments = [attachment]
        contentHandler(content.copy() as! UNNotificationContent)
    }
    
    override func serviceExtensionTimeWillExpire() {
        // Called just before the extension will be terminated by the system.
        // Use this as an opportunity to deliver your "best attempt" at modified content, otherwise the original push payload will be used.
        if let contentHandler = contentHandler, let bestAttemptContent =  bestAttemptContent {
            contentHandler(bestAttemptContent)
        }
    }
    
}

extension UNNotificationAttachment {
    static func create(imageFileIdentifier: String, data: NSData, options: [NSObject : AnyObject]?) -> UNNotificationAttachment? {
        
        let fileManager = FileManager.default
        let tmpSubFolderName = ProcessInfo.processInfo.globallyUniqueString
        let fileURLPath      = NSURL(fileURLWithPath: NSTemporaryDirectory())
        let tmpSubFolderURL  = fileURLPath.appendingPathComponent(tmpSubFolderName, isDirectory: true)
        
        do {
            try fileManager.createDirectory(at: tmpSubFolderURL!, withIntermediateDirectories: true, attributes: nil)
            let fileURL = tmpSubFolderURL?.appendingPathComponent(imageFileIdentifier)
            try data.write(to: fileURL!, options: [])
            let imageAttachment = try UNNotificationAttachment.init(identifier: imageFileIdentifier, url: fileURL!, options: options)
            return imageAttachment
        } catch let error {
            print("error \(error)")
        }
        
        return nil
    }
}

Also make changes in app delegate file accordingly.

