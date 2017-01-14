---
layout: post
title:  "android implementing greenrobot greendao ORM for your sqlite"
date:   2017-01-16 07:07:19
categories: [android, sqlite, database]
comments: true
---



I have been a fan of Firebase since it was launched and started using this wonderful real-time database platform fully since May 2016 for mobile applications I developed.
Though there are some limitations that i have noticed, For example triggering a app to app notification. So we will be implementing a firebase app to app notification without implementing a server backend.
<!--more-->

### What Firebase Provides
Currently via the firebase console all you can do is send push notification to devices only from the console.

![Firebase Console Image](/img/console_notification.png)


### Limitation
We can't programmatically trigger a push notification to a particular device.


### What we want to achieve
We want to be able to notify another device directly of a new notification without having to implement a server side backend to send push notification. So we will be using the firebase database to achieve that

### Steps to achieve this 
* We are going to create a notifications node on our firebase database
* Every user will listen to their notifications node for on child_added events. That is we will have /notifications/user_id/ (notification-objects)
* Then write a service in our android app that listens to the logged in user's notification node for new data
* When a new notification is added against the user's id we then show an android notification to alert the user on the app.

See firebase structure below

![Firebase Console Image](/img/notification_node.png)




So lets start by creating a notification object and then a  method that handles adding of notification to the notifications node. Please note let this method be accessible across your entire app to avoid duplication


{% highlight java %}

public class Notification {

    String user_id;

    public String getUser_id() {
        return user_id;
    }

    public void setUser_id(String user_id) {
        this.user_id = user_id;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public long getTimestamp() {
        return timestamp;
    }

    public void setTimestamp(long timestamp) {
        this.timestamp = timestamp;
    }

    public long getStatus() {
        return status;
    }

    public void setStatus(long status) {
        this.status = status;
    }

    String message;
    String description;
    String type;
    long timestamp,status;

    public Notification() {
    }


    @Exclude
    public Map<String, Object> toMap() {
        HashMap<String, Object> result = new HashMap<>();
        result.put("message", message);
        result.put("description", description);
        result.put("timestamp", ServerValue.TIMESTAMP);
        result.put("type",type);
        result.put("status",status);
        return result;
    }
}

{% endhighlight java %}

And then we create the method as seen below

{% highlight java %}
   public static void sendNotification(String user_id,String message,String description,String type){
        DatabaseReference databaseReference = FirebaseDatabase.getInstance().getReference("notifications").child(user_id);
        String pushKey = databaseReference.push().getKey();

        Notification notification = new Notification();
        notification.setDescription(description);
        notification.setMessage(message);
        notification.setUser_id(user_id);
        notification.setType(type);

        Map<String, Object> forumValues = notification.toMap();
        Map<String, Object> childUpdates = new HashMap<>();
        childUpdates.put(pushKey, forumValues);
        databaseReference.setPriority(ServerValue.TIMESTAMP);
        databaseReference.updateChildren(childUpdates, new DatabaseReference.CompletionListener() {
            @Override
            public void onComplete(DatabaseError databaseError, DatabaseReference databaseReference) {
                if(databaseError == null){

                }
            }
        });
    }
{% endhighlight %}


So calling this method above and passing the necessary parameters. And <b> please note </b> Verify the logged in user is not the sender so as to avoid sending notification to same user.

{% highlight java %}

  sendNotification(
            receiver_id, /*who the notification is meant for*/
            "Chat message from " /*Message to be displayed on the notification*/
            "New chat message", /*Message title*/
            "chat_view" /*Notification type, You can use this to determine what activities to stack when the receiver clicks on the notification item*/
   );

{% endhighlight java %}


### Implementing a Notification Service
So what is next is to create our notification service. Ensure this service is running at all time.


To avoid multiple notification we have to update the notification status to 1 after firing a notification

{% highlight java %}

public class FirebaseNotificationServices extends Service{

    public FirebaseDatabase mDatabase;
    FirebaseAuth firebaseAuth;
    Context context;
    static String TAG = "FirebaseService";

    @Override
    public void onCreate() {
        super.onCreate();
        context = this;



        mDatabase = FirebaseDatabase.getInstance();
        firebaseAuth = FirebaseAuth.getInstance();

        setupNotificationListener();
    }



    private void setupNotificationListener() {

        mDatabase.getReference().child("notifications")
                .child(firebaseAuth.getCurrentUser().getUid())
                .orderByChild("status").equalTo(0)
                .addChildEventListener(new ChildEventListener() {
            @Override
            public void onChildAdded(DataSnapshot dataSnapshot, String s) {
                if(dataSnapshot != null){
                    Notification notification = dataSnapshot.getValue(Notification.class);

                    showNotification(context,notification,dataSnapshot.getKey());
                }
            }

            @Override
            public void onChildChanged(DataSnapshot dataSnapshot, String s) {
                Utilities.log("onChildChanged",dataSnapshot);
            }

            @Override
            public void onChildRemoved(DataSnapshot dataSnapshot) {
                Utilities.log("onChildRemoved",dataSnapshot);
            }

            @Override
            public void onChildMoved(DataSnapshot dataSnapshot, String s) {
                Utilities.log("onChildMoved",dataSnapshot);

            }

            @Override
            public void onCancelled(DatabaseError databaseError) {
                Utilities.log("onCancelled",databaseError);
            }
        });


    }

 
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        return Service.START_STICKY;
    }

    @Override
    public IBinder onBind(Intent intent) {
        // TODO: Return the communication channel to the service.
        throw new UnsupportedOperationException("Not yet implemented");
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
    }

    private void showNotification(Context context, Notification notification,String notification_key){
        NotificationCompat.Builder mBuilder = new NotificationCompat.Builder(context)
                .setSmallIcon(R.mipmap.ic_launcher)
                .setContentTitle(notification.getDescription())
                .setDefaults(NotificationCompat.DEFAULT_ALL)
                .setContentText(Html.fromHtml(notification.getMessage()
                ))
                .setAutoCancel(true);

        Intent backIntent = new Intent(context, MainActivity.class);
        backIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);

        Intent intent = new Intent(context, FriendsView.class);

        /*  Use the notification type to switch activity to stack on the main activity*/
        if(notification.getType().equals("chat_view")){
            intent = new Intent(context, FriendsView.class);
        }


        final PendingIntent pendingIntent = PendingIntent.getActivities(context, 900,
                new Intent[] {backIntent}, PendingIntent.FLAG_ONE_SHOT);


        TaskStackBuilder stackBuilder = TaskStackBuilder.create(context);
        stackBuilder.addParentStack(MainActivity.class);

        mBuilder.setContentIntent(pendingIntent);


        NotificationManager mNotificationManager =  (NotificationManager)context. getSystemService(Context.NOTIFICATION_SERVICE);
        mNotificationManager.notify(1, mBuilder.build());

        /* Update firebase set notifcation with this key to 1 so it doesnt get pulled by our notification listener*/
        flagNotificationAsSent(notification_key);
    }

    private void flagNotificationAsSent(String notification_key) {
        mDatabase.getReference().child("notifications")
                .child(firebaseAuth.getCurrentUser().getUid())
                .child(notification_key)
                .child("status")
                .setValue(1);
    }

}
{% endhighlight %}



### Full Project
Access full project here [Github Repo](https://github.com/akinmobile/Firebase-Device-Notification) 




And that's all. We can now make an app to app notification using firebase without implementing a server backend.

I hope this is useful if you have any issues at all please feel free to drop a comment.

Cheers <br>
Happy coding.

