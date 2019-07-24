#  inBrain-Android ![API](https://img.shields.io/badge/API-16%2B-brightgreen.svg?style=flat) 
## Integration
Integration is easiest with gradle dependency. Just add the following implementation line into dependencies section of your build.gradle file (app module).
```groovy
dependencies {  
    implementation 'com.github.inbrainai:sdk-android:0.1.4'  
}
```
## Configure inBrain SDK
First of all you need to initialize the SDK using the following call:
```
InBrainCallback callback = new InBrainCallback() {
    @Override
    public void onAdClosed() {
    }
    
    // Notifies your application about new rewards.
    @Override  
    public boolean handleRewards(List<Reward> rewards) {
        // Process received rewards here
        return true; // if processed successfully, false otherwise
    }  
};
InBrain.getInstance().init(this, CLIENT_ID, CLIENT_SECRET, callback);
```

`CLIENT_ID` - your client ID obtained from your account manager.

`CLIENT_SECRET` - your client secret obtained from your account manager.
This should better be done just once. In order to do that you can add initialization code into `onCreate` method of your Application class code.

`handleRewards` method of the callback should return true if received rewards were processed successfully (added to user's balance), false otherwise. If you return false, the same rewards will be passed to you again later so you could retry the operation.

## Set user ID
`InBrain.getInstance().setAppUserId(USER_ID);` where `USER_ID` is the unique identifier of the current user. This should be done when you already know how to identify the current user, usually that happens after sign-up/sign-in process.

## Presenting the survey wall
In order to open the inBrain survey wall execute the following call:
```
InBrain.getInstance().showSurveys(callback);
```
This will open the survey wall in new activity. inBrain SDK will handle everything else. It will return control to last opened activity of your app after user leaves the survey wall.

## InBrainCallback
Received rewards will be passed to `handleRewards(List<Reward> rewards)` method of your callback.

Rewards that have been processed need to be confirmed with InBrain so it would not pass it back to the app again and again. You have two options to confirm rewards:

 * Simple synchronous method: just `return true` from `handleRewards` in the callback. This will confirm rewards instantly.
 * Advanced asynchronous method: `return false` from `handleRewards` method. Later, after you have processed the rewards, make a call to `confirmRewards` method. The call will look like this: `InBrain.getInstance().confirmRewards(list)`, where `list` is the list of rewards you want to confirm. All unconfirmed rewards will be passed again to `handleRewards` on subsequent calls.
 
 **If you use advanced method, make sure to confirm received rewards to avoid duplicate rewards!**

`onAdClosed` method is called when InBrain screen is closed & user returns back to your app.

## Request rewards manually
Once in a while new rewards will be passed to your app through `handleRewards` calls of your `InBrainCallback` instances. This mechanism is very reliable and it is usually enough for most apps. However if you want to check for new rewards at a certain time, there is a special method which invokes manual check:
```java
InBrain.getInstance().getRewards(new GetRewardsCallback() {  
    @Override  
    public boolean handleRewards(List<Reward> rewards) {  
        // Process rewards
        return true; // or return false and confirm your rewards manualy using InBrain.getInstance().confirmRewards(rewards);  
    }  
  
    @Override  
  public void onFailToLoadRewards(int errorCode) {  
    }  
});
```
