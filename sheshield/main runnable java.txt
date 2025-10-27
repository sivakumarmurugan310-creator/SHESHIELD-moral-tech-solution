package com.example.sheshield;

import android.Manifest;
import android.content.Intent;
import android.content.pm.PackageManager;
import android.location.Location;
import android.location.LocationListener;
import android.location.LocationManager;
import android.media.MediaPlayer;
import android.media.MediaRecorder;
import android.net.Uri;
import android.os.Bundle;
import android.provider.MediaStore;
import android.telephony.SmsManager;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import java.io.File;
import java.io.IOException;
import java.util.Locale;

public class MainActivity extends AppCompatActivity {

    private EditText editContacts;
    private Button btnSOS, btnRecordAudio, btnRecordVideo, btnDangerZone;

    private MediaRecorder audioRecorder;
    private boolean isRecording = false;
    private String audioFilePath;

    private static final int REQUEST_PERMISSIONS = 200;
    private static final int VIDEO_REQUEST_CODE = 300;

    private LocationManager locationManager;
    private String currentLocation = "Unknown";

    private MediaPlayer alarmPlayer;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        editContacts = findViewById(R.id.editContacts);
        btnSOS = findViewById(R.id.btnSOS);
        btnRecordAudio = findViewById(R.id.btnRecordAudio);
        btnRecordVideo = findViewById(R.id.btnRecordVideo);
        btnDangerZone = findViewById(R.id.btnDangerZone);

        // Permissions
        String[] permissions = {
                Manifest.permission.SEND_SMS,
                Manifest.permission.RECORD_AUDIO,
                Manifest.permission.CAMERA,
                Manifest.permission.ACCESS_FINE_LOCATION
        };
        ActivityCompat.requestPermissions(this, permissions, REQUEST_PERMISSIONS);

        // Location Manager
        locationManager = (LocationManager) getSystemService(LOCATION_SERVICE);

        // Alarm sound
        alarmPlayer = MediaPlayer.create(this, R.raw.alarm_sound);

        // Listeners
        btnSOS.setOnClickListener(v -> triggerSOS());
        btnRecordAudio.setOnClickListener(v -> toggleAudioRecording());
        btnRecordVideo.setOnClickListener(v -> recordVideo());
        btnDangerZone.setOnClickListener(v -> simulateDangerZone());
    }

    // 🔴 SOS = SMS + Location + Audio + Video + Danger Zone
    private void triggerSOS() {
        getLocation();
        sendSOS();
        startAudioRecording();
        recordVideo();
        simulateDangerZone();
    }

    // 📩 Send SMS to 3 trusted contacts
    private void sendSOS() {
        String[] contacts = {
                "9790308661", // Trusted contact 1
                "9876543210", // Trusted contact 2
                "9123456780"  // Trusted contact 3
        };

        String message = "🚨 SOS! I need help. My location: https://maps.google.com/?q=" + currentLocation;

        try {
            SmsManager smsManager = SmsManager.getDefault();
            for (String contact : contacts) {
                smsManager.sendTextMessage(contact.trim(), null, message, null, null);
            }
            Toast.makeText(this, "SOS sent to all contacts!", Toast.LENGTH_SHORT).show();
        } catch (Exception e) {
            Toast.makeText(this, "Failed to send SOS: " + e.getMessage(), Toast.LENGTH_LONG).show();
        }
    }

    // 🎙️ Audio Recording
    private void toggleAudioRecording() {
        if (isRecording) {
            stopAudioRecording();
        } else {
            startAudioRecording();
        }
    }

    private void startAudioRecording() {
        File folder = new File(getExternalFilesDir(null), "SheShieldRecordings");
        if (!folder.exists()) folder.mkdirs();

        audioFilePath = folder.getAbsolutePath() + "/audio_" + System.currentTimeMillis() + ".3gp";

        audioRecorder = new MediaRecorder();
        audioRecorder.setAudioSource(MediaRecorder.AudioSource.MIC);
        audioRecorder.setOutputFormat(MediaRecorder.OutputFormat.THREE_GPP);
        audioRecorder.setOutputFile(audioFilePath);
        audioRecorder.setAudioEncoder(MediaRecorder.AudioEncoder.AMR_NB);

        try {
            audioRecorder.prepare();
            audioRecorder.start();
            isRecording = true;
            Toast.makeText(this, "Audio recording started", Toast.LENGTH_SHORT).show();
        } catch (IOException e) {
            Toast.makeText(this, "Recording failed: " + e.getMessage(), Toast.LENGTH_LONG).show();
        }
    }

    private void stopAudioRecording() {
        if (audioRecorder != null) {
            audioRecorder.stop();
            audioRecorder.release();
            audioRecorder = null;
            isRecording = false;
            Toast.makeText(this, "Audio saved: " + audioFilePath, Toast.LENGTH_LONG).show();
        }
    }

    // 📹 Video Recording
    private void recordVideo() {
        Intent intent = new Intent(MediaStore.ACTION_VIDEO_CAPTURE);
        startActivityForResult(intent, VIDEO_REQUEST_CODE);
    }

    // 📍 Get Location
    private void getLocation() {
        if (ActivityCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
            return;
        }

        Location lastLocation = locationManager.getLastKnownLocation(LocationManager.GPS_PROVIDER);
        if (lastLocation != null) {
            currentLocation = String.format(Locale.ENGLISH, "%.6f,%.6f",
                    lastLocation.getLatitude(), lastLocation.getLongitude());
        }

        locationManager.requestLocationUpdates(LocationManager.GPS_PROVIDER, 5000, 5, new LocationListener() {
            @Override
            public void onLocationChanged(@NonNull Location location) {
                currentLocation = String.format(Locale.ENGLISH, "%.6f,%.6f",
                        location.getLatitude(), location.getLongitude());
            }
        });
    }

    // ⚠️ Danger Zone Alert
    private void simulateDangerZone() {
        Toast.makeText(this, "⚠️ Danger Zone detected! Suggesting safe path...", Toast.LENGTH_LONG).show();
        playAlarm();
        suggestSafeRoute();
    }

    private void playAlarm() {
        if (alarmPlayer != null) {
            alarmPlayer.start();
        }
    }

    private void suggestSafeRoute() {
        // Open Google Maps to nearest Police Station
        Uri gmmIntentUri = Uri.parse("geo:0,0?q=police station");
        Intent mapIntent = new Intent(Intent.ACTION_VIEW, gmmIntentUri);
        mapIntent.setPackage("com.google.android.apps.maps");
        startActivity(mapIntent);
    }

    // ✅ Permissions Result
    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions,
                                           @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if (requestCode == REQUEST_PERMISSIONS) {
            for (int result : grantResults) {
                if (result != PackageManager.PERMISSION_GRANTED) {
                    Toast.makeText(this, "Please grant all permissions!", Toast.LENGTH_LONG).show();
                    return;
                }
            }
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (alarmPlayer != null) {
            alarmPlayer.release();
        }
    }
}
