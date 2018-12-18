...menustart

 - [Create Instant Games](#009069c69b11caef6d75739ab207e856)
 - [Create Message Bot](#1b3b0682a7aea56937be67c5a729a846)
 - [debug messager bot with local server](#aceacf6ca899a80c0a70aa4f2522d77c)

...menuend


<h2 id="009069c69b11caef6d75739ab207e856"></h2>

# Create Instant Games

 1. create a new app
 2. set as game 
    - /Setting / basic / Category
 3. 为应用添加管理员、开发者和测试员
    - Facebook 正在逐步面向更广泛的受众推广小游戏，您的游戏目前可能无法对所有用户开放。
    - 要确保游戏的所有发开者和测试员拥有访问权限，请在应用面板的**/Roles**选项卡中将他们添加为应用的管理员、开发者或测试员
 4. Enable Instant Games in the App Dashboard
    - PRODUCTS +  > instant game > Set up
    - Make sure to enable the the **Use Instant Games** toggle.
 5. Testing and Uploading
    - To upload the .zip file, click the **Web Hosting** tab in the App Dashboard
    - and click **+Upload Version**  to upload the .zip file to Facebook's hosting service 
    - After that, the build will process the file, which should only take a few seconds. When the state changes to "Standby", click the **"★"** button to push the build to production.
 6. Setting up a Game Bot
    - Step 1: Create a Page,  
        - it needs some some special properties:
            - The page's category needs to be **App Page**
            - The page's name needs to **contain the name** of the app.
            - The page **cannot be associated** with another app.
        - Instant Games/Details/App Page
    - Step 2: Activate your Bot 
        - first create a message bot ( see below )
        - Instant Games bots are only permitted to use standard messaging and the GAME_EVENT message tag but not pages_messaging_subscriptions.
    - Step 3: https://developers.facebook.com/docs/games/instant-games/guides/bots-and-server-communication/
 7. https://developers.facebook.com/docs/games/instant-games/guides/bots-and-server-communication/
    - 客户端发送 sessionData , 游戏关闭时，这个sessionData 会作为 `payload` property  of the game_play webhook.

```
FBInstant.setSessionData({
  scoutSent:true,
  scoutDurationInHours:24
});
```



<h2 id="1b3b0682a7aea56937be67c5a729a846"></h2>

# Create Message Bot

https://developers.facebook.com/docs/messenger-platform/getting-started


 1. Add the Messenger Platform to your Facebook app
    - 'PRODUCTS', click '+ Add Product' / 'Messenger'  /  'Set Up' 
 2. Configure the webhook for your app
    - In the 'Webhooks' section of the Messenger settings console , 'Setup Webhooks' 
    - 'Callback URL' : something like `https://domain/webhook`
    - 'Verify Token' : your custom token
    - 'Subscription Fields': `messages` and `messaging_postbacks`
 3. Subscribe your app to a Facebook Page
    - In the 'Token Generation' section , choose page, get the Page Access Token
        - 每选择 一次page，就会重新生成 token
    - In the 'Webhook' section ,  select the same page , Click the 'Subscribe' to  subscribe your app to receive webhook events for the Page.
 4. Test your app subscription
    - send a message to your Page from facebook.com or in Messenger. 
    - If your webhook receives a webhook event, you have fully set up your app!


<h2 id="aceacf6ca899a80c0a70aa4f2522d77c"></h2>

# debug messager bot with local server

 - localtunnel or ngrok

