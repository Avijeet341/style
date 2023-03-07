package com.avi.notifications

import android.Manifest
import android.annotation.SuppressLint
import android.app.Notification
import android.app.NotificationChannel
import android.app.NotificationManager
import android.app.PendingIntent
import android.content.Context
import android.content.Intent
import android.content.pm.PackageManager
import android.graphics.Bitmap
import android.graphics.BitmapFactory
import android.os.Build
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import androidx.core.app.ActivityCompat
import androidx.core.app.NotificationCompat
import androidx.core.app.NotificationManagerCompat
import com.avi.notifications.databinding.ActivityMainBinding

class MainActivity : AppCompatActivity() {
    private val CHANNEL_ID="1"
    lateinit var mainBinding: ActivityMainBinding

     var counter=0
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        mainBinding=ActivityMainBinding.inflate(layoutInflater)
        val view=mainBinding.root
        setContentView(view)
        mainBinding.sendNotification.setOnClickListener {
            counter++
            mainBinding.sendNotification.text=counter.toString()
            if(counter%5==0){
                startNotification()
            }
        }

    }
    @SuppressLint("SuspiciousIndentation")
    fun  startNotification(){

        val intent=Intent(applicationContext,MainActivity::class.java)

        val pendingIntent=if(Build.VERSION.SDK_INT>=23){PendingIntent.getActivity(applicationContext,0,intent,PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE)}
        else{
            PendingIntent.getActivity(applicationContext,0,intent,PendingIntent.FLAG_UPDATE_CURRENT)
        }
        // Action button like toast message and dismiss btn


        val actionIntent=Intent(applicationContext,ReceiverAvi::class.java)
            actionIntent.putExtra("toast","This is a notification message.")

        val actionPendingIntent=if(Build.VERSION.SDK_INT>=23){PendingIntent.getBroadcast(applicationContext,1,actionIntent,PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE)}
        else{
            PendingIntent.getBroadcast(applicationContext,1,actionIntent,PendingIntent.FLAG_UPDATE_CURRENT)
        }

        //dismissButton
        val dismissActionIntent=Intent(applicationContext,ReceiverDismiss::class.java)

        val dismissActionPendingIntent=if(Build.VERSION.SDK_INT>=23){PendingIntent.getBroadcast(applicationContext,2,dismissActionIntent,PendingIntent.FLAG_UPDATE_CURRENT or PendingIntent.FLAG_IMMUTABLE)}
        else{
            PendingIntent.getBroadcast(applicationContext,2,dismissActionIntent,PendingIntent.FLAG_UPDATE_CURRENT)
        }



//        image and long text
         val icon: Bitmap = BitmapFactory.decodeResource(resources,R.drawable.android)
         val text:String=resources.getString(R.string.big_text)

        val builder=NotificationCompat.Builder(this@MainActivity,CHANNEL_ID)
        if(Build.VERSION.SDK_INT>=Build.VERSION_CODES.O){
            val channel=NotificationChannel(CHANNEL_ID,"1",NotificationManager.IMPORTANCE_DEFAULT)

            val manager:NotificationManager=getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
            manager.createNotificationChannel(channel)

            builder.setSmallIcon(R.drawable.small_icon)
                .setContentTitle("Title")
                .setContentText("Notification Text")
                .setContentIntent(pendingIntent)
                .setAutoCancel(true)//false notification tag will automatically be gone after you clked it if true
                .addAction(R.drawable.small_icon,"Toast message",actionPendingIntent)
                .addAction(R.drawable.small_icon,"Dismiss",dismissActionPendingIntent)
                .setLargeIcon(icon)
                .setStyle(NotificationCompat.BigPictureStyle().bigLargeIcon(icon))
//                .setStyle(NotificationCompat.BigTextStyle().bigText(text))

        }
        else{
            builder.setSmallIcon(R.drawable.small_icon)
                .setContentTitle("Title")
                .setContentText("This is the notification text")
                .setPriority(NotificationCompat.PRIORITY_DEFAULT)
                .setContentIntent(pendingIntent)
                .setAutoCancel(true)
                .addAction(R.drawable.small_icon,"Toast message",actionPendingIntent)
                .addAction(R.drawable.small_icon,"Dismiss",dismissActionPendingIntent)
                .setLargeIcon(icon)
                .setStyle(NotificationCompat.BigPictureStyle().bigLargeIcon(icon))
//                .setStyle(NotificationCompat.BigTextStyle().bigText(text))
        }
        val notificationManagerCompat=NotificationManagerCompat.from(this@MainActivity)

        if (ActivityCompat.checkSelfPermission(
                this,
                Manifest.permission.POST_NOTIFICATIONS
            ) != PackageManager.PERMISSION_GRANTED
        ) {
            // TODO: Consider calling
            //    ActivityCompat#requestPermissions
            // here to request the missing permissions, and then overriding
            //   public void onRequestPermissionsResult(int requestCode, String[] permissions,
            //                                          int[] grantResults)
            // to handle the case where the user grants the permission. See the documentation
            // for ActivityCompat#requestPermissions for more details.
            return
        }
        notificationManagerCompat.notify(1,builder.build())
    }
}
