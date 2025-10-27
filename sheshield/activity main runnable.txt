<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="20dp"
    android:background="#f5f5f5">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="SheShield - Women's Safety App"
        android:textSize="20sp"
        android:textStyle="bold"
        android:layout_gravity="center"
        android:layout_marginBottom="20dp"/>

    <EditText
        android:id="@+id/editContacts"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter trusted contacts (+91...), separated by commas"
        android:padding="10dp"
        android:background="@android:drawable/edit_text"
        android:layout_marginBottom="15dp"/>

    <Button
        android:id="@+id/btnSOS"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Send SOS"
        android:layout_marginBottom="15dp"/>

    <Button
        android:id="@+id/btnRecordAudio"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Record Audio"
        android:layout_marginBottom="15dp"/>

    <Button
        android:id="@+id/btnRecordVideo"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Record Video"
        android:layout_marginBottom="15dp"/>

    <Button
        android:id="@+id/btnDangerZone"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Simulate Danger Zone"
        android:layout_marginBottom="15dp"/>

</LinearLayout>
