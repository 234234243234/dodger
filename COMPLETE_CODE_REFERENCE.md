# Brawl Stars Auto Dodge - Complete Code Reference

This file contains all the code for the Brawl Stars Auto Dodge app in a single document for easy reference. Note that to actually build the APK, you'll need the proper project structure with all files in their correct locations.

## MainActivity.kt

```kotlin
package com.autododge.brawlstars

import android.content.Intent
import android.media.projection.MediaProjectionManager
import android.net.Uri
import android.os.Bundle
import android.provider.Settings
import android.view.View
import android.widget.Button
import android.widget.TextView
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.core.content.ContextCompat

class MainActivity : AppCompatActivity() {
    companion object {
        private const val REQUEST_MEDIA_PROJECTION = 100
        private const val REQUEST_OVERLAY_PERMISSION = 101
        private const val REQUEST_ACCESSIBILITY_SETTINGS = 102
    }

    private lateinit var startServiceButton: Button
    private lateinit var permissionStatusText: TextView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        startServiceButton = findViewById(R.id.startServiceButton)
        permissionStatusText = findViewById(R.id.permissionStatusText)

        startServiceButton.setOnClickListener {
            checkAndRequestPermissions()
        }

        updatePermissionStatus()
    }

    private fun updatePermissionStatus() {
        val canDrawOverlays = Settings.canDrawOverlays(this)
        val accessibilityEnabled = isAccessibilityServiceEnabled()

        val overlayStatus = if (canDrawOverlays) "✅" else "❌"
        val accessibilityStatus = if (accessibilityEnabled) "✅" else "❌"

        permissionStatusText.text = "Overlay permission: $overlayStatus\n" +
                "Accessibility permission: $accessibilityStatus"

        startServiceButton.isEnabled = canDrawOverlays && accessibilityEnabled
    }

    private fun isAccessibilityServiceEnabled(): Boolean {
        val accessibilityEnabled = try {
            Settings.Secure.getInt(contentResolver, Settings.Secure.ACCESSIBILITY_ENABLED)
        } catch (e: Settings.SettingNotFoundException) {
            0
        }

        if (accessibilityEnabled == 1) {
            val serviceString = Settings.Secure.getString(contentResolver, Settings.Secure.ENABLED_ACCESSIBILITY_SERVICES)
            serviceString?.let {
                return it.contains("${packageName}/${packageName}.AutoDodgeService")
            }
        }
        return false
    }

    private fun checkAndRequestPermissions() {
        // Check for overlay permission
        if (!Settings.canDrawOverlays(this)) {
            requestOverlayPermission()
            return
        }

        // Check for accessibility permission
        if (!isAccessibilityServiceEnabled()) {
            requestAccessibilityPermission()
            return
        }

        // Request screen capture permission
        requestScreenCapturePermission()
    }

    private fun requestOverlayPermission() {
        val intent = Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION, Uri.parse("package:$packageName"))
        startActivityForResult(intent, REQUEST_OVERLAY_PERMISSION)
        Toast.makeText(this, "Please grant overlay permission", Toast.LENGTH_LONG).show()
    }

    private fun requestAccessibilityPermission() {
        val intent = Intent(Settings.ACTION_ACCESSIBILITY_SETTINGS)
        startActivityForResult(intent, REQUEST_ACCESSIBILITY_SETTINGS)
        Toast.makeText(this, "Please enable AutoDodge accessibility service", Toast.LENGTH_LONG).show()
    }

    private fun requestScreenCapturePermission() {
        val mediaProjectionManager = getSystemService(MEDIA_PROJECTION_SERVICE) as MediaProjectionManager
        startActivityForResult(mediaProjectionManager.createScreenCaptureIntent(), REQUEST_MEDIA_PROJECTION)
    }

    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)

        when (requestCode) {
            REQUEST_MEDIA_PROJECTION -> {
                if (resultCode == RESULT_OK && data != null) {
                    startOverlayService(data)
                } else {
                    Toast.makeText(this, "Screen capture permission denied", Toast.LENGTH_SHORT).show()
                }
            }
            REQUEST_OVERLAY_PERMISSION, REQUEST_ACCESSIBILITY_SETTINGS -> {
                updatePermissionStatus()
            }
        }
    }

    private fun startOverlayService(screenCaptureIntent: Intent) {
        // Start the overlay service with screen capture intent
        val intent = Intent(this, OverlayService::class.java).apply {
            putExtra("resultCode", RESULT_OK)
            putExtra("data", screenCaptureIntent)
        }
        ContextCompat.startForegroundService(this, intent)
        
        Toast.makeText(this, "Auto Dodge service started", Toast.LENGTH_SHORT).show()
        moveTaskToBack(true)
    }

    override fun onResume() {
        super.onResume()
        updatePermissionStatus()
    }
}
```

## AutoDodgeService.kt

```kotlin
package com.autododge.brawlstars

import android.accessibilityservice.AccessibilityService
import android.accessibilityservice.GestureDescription
import android.content.Intent
import android.graphics.Path
import android.os.Handler
import android.os.Looper
import android.view.accessibility.AccessibilityEvent
import android.widget.Toast
import com.autododge.brawlstars.utils.GestureUtils

class AutoDodgeService : AccessibilityService() {
    
    companion object {
        private var instance: AutoDodgeService? = null
        
        fun getInstance(): AutoDodgeService? {
            return instance
        }
    }
    
    private val mainHandler = Handler(Looper.getMainLooper())
    
    override fun onServiceConnected() {
        super.onServiceConnected()
        instance = this
        
        Toast.makeText(this, "Auto Dodge Accessibility Service connected", Toast.LENGTH_SHORT).show()
    }
    
    override fun onAccessibilityEvent(event: AccessibilityEvent?) {
        // Not needed for our use case
    }
    
    override fun onInterrupt() {
        // Not needed for our use case
    }
    
    override fun onUnbind(intent: Intent?): Boolean {
        instance = null
        return super.onUnbind(intent)
    }
    
    /**
     * Executes a dodge movement in the specified direction
     * 
     * @param direction Direction to dodge (0-360 degrees, 0 is right, 90 is up)
     * @param distance Distance to move in pixels
     */
    fun executeDodge(direction: Float, distance: Float, screenWidth: Int, screenHeight: Int) {
        // Calculate the center of the screen (likely where game controls are)
        val centerX = screenWidth / 2f
        val centerY = screenHeight * 0.75f  // Assuming controls are in bottom half
        
        // Calculate end point based on direction and distance
        val angleRadians = Math.toRadians(direction.toDouble())
        val endX = centerX + (Math.cos(angleRadians) * distance).toFloat()
        val endY = centerY - (Math.sin(angleRadians) * distance).toFloat()
        
        // Create path for gesture
        val path = Path().apply {
            moveTo(centerX, centerY)
            lineTo(endX, endY)
        }
        
        // Execute gesture
        val gestureDescription = GestureUtils.createGesture(path, 0, 200)
        dispatchGesture(gestureDescription, null, null)
    }
    
    /**
     * Execute a tap at specific coordinates
     */
    fun executeTap(x: Float, y: Float) {
        val path = Path().apply {
            moveTo(x, y)
            lineTo(x, y)
        }
        
        val gestureDescription = GestureUtils.createGesture(path, 0, 100)
        dispatchGesture(gestureDescription, null, null)
    }
}
```

## OverlayService.kt

```kotlin
package com.autododge.brawlstars

import android.app.Notification
import android.app.NotificationChannel
import android.app.NotificationManager
import android.app.Service
import android.content.Context
import android.content.Intent
import android.graphics.Bitmap
import android.graphics.PixelFormat
import android.media.projection.MediaProjection
import android.media.projection.MediaProjectionManager
import android.os.Build
import android.os.Handler
import android.os.IBinder
import android.os.Looper
import android.view.Gravity
import android.view.LayoutInflater
import android.view.View
import android.view.WindowManager
import android.widget.Button
import android.widget.CompoundButton
import android.widget.Switch
import android.widget.Toast
import androidx.core.app.NotificationCompat
import com.autododge.brawlstars.utils.ScreenCaptureManager

class OverlayService : Service() {

    companion object {
        private const val NOTIFICATION_ID = 1
        private const val CHANNEL_ID = "AutoDodgeChannel"
        private const val SCREEN_CAPTURE_FPS = 30L // frames per second
    }

    private lateinit var windowManager: WindowManager
    private lateinit var overlayView: View
    private lateinit var projectileDetector: ProjectileDetector
    private lateinit var dodgeController: DodgeController
    private lateinit var screenCaptureManager: ScreenCaptureManager
    
    private val handler = Handler(Looper.getMainLooper())
    private var isServiceRunning = false
    private var isAutoDodgeEnabled = false

    override fun onCreate() {
        super.onCreate()
        
        createNotificationChannel()
        
        val notification = createNotification()
        startForeground(NOTIFICATION_ID, notification)
        
        projectileDetector = ProjectileDetector()
        
        windowManager = getSystemService(WINDOW_SERVICE) as WindowManager
        initOverlay()
    }

    private fun createNotificationChannel() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                CHANNEL_ID,
                "Auto Dodge Service",
                NotificationManager.IMPORTANCE_LOW
            ).apply {
                description = "Notification for Auto Dodge Service"
            }
            
            val notificationManager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
            notificationManager.createNotificationChannel(channel)
        }
    }

    private fun createNotification(): Notification {
        return NotificationCompat.Builder(this, CHANNEL_ID)
            .setContentTitle("Auto Dodge")
            .setContentText("Auto Dodge service is running")
            .setSmallIcon(android.R.drawable.ic_menu_compass)
            .setPriority(NotificationCompat.PRIORITY_LOW)
            .build()
    }

    private fun initOverlay() {
        val inflater = getSystemService(Context.LAYOUT_INFLATER_SERVICE) as LayoutInflater
        overlayView = inflater.inflate(R.layout.overlay_controls, null)
        
        val params = WindowManager.LayoutParams(
            WindowManager.LayoutParams.WRAP_CONTENT,
            WindowManager.LayoutParams.WRAP_CONTENT,
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) 
                WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY 
            else 
                WindowManager.LayoutParams.TYPE_PHONE,
            WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE,
            PixelFormat.TRANSLUCENT
        ).apply {
            gravity = Gravity.TOP or Gravity.END
        }
        
        val closeButton = overlayView.findViewById<Button>(R.id.closeButton)
        val autoDodgeSwitch = overlayView.findViewById<Switch>(R.id.autoDodgeSwitch)
        
        closeButton.setOnClickListener {
            stopSelf()
        }
        
        autoDodgeSwitch.setOnCheckedChangeListener { _, isChecked ->
            isAutoDodgeEnabled = isChecked
            
            if (isChecked && !isServiceRunning) {
                startScreenCapture()
            }
            
            Toast.makeText(
                this, 
                if (isChecked) "Auto dodge enabled" else "Auto dodge disabled", 
                Toast.LENGTH_SHORT
            ).show()
        }
        
        windowManager.addView(overlayView, params)
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        if (intent != null) {
            val resultCode = intent.getIntExtra("resultCode", 0)
            val data = intent.getParcelableExtra<Intent>("data")
            
            if (resultCode != 0 && data != null) {
                val mediaProjectionManager = getSystemService(Context.MEDIA_PROJECTION_SERVICE) as MediaProjectionManager
                val mediaProjection = mediaProjectionManager.getMediaProjection(resultCode, data)
                
                if (mediaProjection != null) {
                    setupScreenCapture(mediaProjection)
                }
            }
        }
        
        return START_STICKY
    }
    
    private fun setupScreenCapture(mediaProjection: MediaProjection) {
        screenCaptureManager = ScreenCaptureManager(this, mediaProjection)
        dodgeController = DodgeController(AutoDodgeService.getInstance())
    }
    
    private fun startScreenCapture() {
        if (!::screenCaptureManager.isInitialized) {
            Toast.makeText(this, "Screen capture not initialized", Toast.LENGTH_SHORT).show()
            return
        }
        
        isServiceRunning = true
        
        val frameInterval = 1000L / SCREEN_CAPTURE_FPS
        
        val captureRunnable = object : Runnable {
            override fun run() {
                if (isAutoDodgeEnabled && isServiceRunning) {
                    screenCaptureManager.captureScreen { bitmap ->
                        processBitmap(bitmap)
                    }
                    
                    handler.postDelayed(this, frameInterval)
                }
            }
        }
        
        handler.post(captureRunnable)
    }
    
    private fun processBitmap(bitmap: Bitmap) {
        // Process the captured screen with projectile detector
        val projectiles = projectileDetector.detectProjectiles(bitmap)
        
        // If projectiles detected, execute dodge
        if (projectiles.isNotEmpty()) {
            val screenWidth = bitmap.width
            val screenHeight = bitmap.height
            
            // Calculate best dodge direction based on projectile positions
            dodgeController.calculateAndExecuteDodge(
                projectiles, 
                screenWidth, 
                screenHeight
            )
        }
    }

    override fun onDestroy() {
        isServiceRunning = false
        isAutoDodgeEnabled = false
        
        if (::windowManager.isInitialized && overlayView.isAttachedToWindow) {
            windowManager.removeView(overlayView)
        }
        
        if (::screenCaptureManager.isInitialized) {
            screenCaptureManager.release()
        }
        
        super.onDestroy()
    }

    override fun onBind(intent: Intent?): IBinder? {
        return null
    }
}
```

## ProjectileDetector.kt

```kotlin
package com.autododge.brawlstars

import android.graphics.Bitmap
import android.graphics.Point
import org.opencv.android.Utils
import org.opencv.core.*
import org.opencv.imgproc.Imgproc
import java.util.*

class ProjectileDetector {

    data class Projectile(
        val position: Point, 
        val velocity: Point, 
        val size: Float, 
        val confidence: Float
    )

    private var previousFrame: Mat? = null
    private var previousProjectiles = mutableListOf<Projectile>()
    private val projectileHistory = LinkedList<List<Projectile>>()
    private val MAX_HISTORY = 5
    
    init {
        try {
            System.loadLibrary("opencv_java4")
        } catch (e: UnsatisfiedLinkError) {
            e.printStackTrace()
        }
    }
    
    fun detectProjectiles(bitmap: Bitmap): List<Projectile> {
        val currentFrame = Mat()
        Utils.bitmapToMat(bitmap, currentFrame)
        
        // Convert to grayscale for processing
        val grayFrame = Mat()
        Imgproc.cvtColor(currentFrame, grayFrame, Imgproc.COLOR_BGR2GRAY)
        
        // Apply Gaussian blur to reduce noise
        Imgproc.GaussianBlur(grayFrame, grayFrame, Size(5.0, 5.0), 0.0)
        
        val detectedProjectiles = mutableListOf<Projectile>()
        
        // If we have a previous frame, detect motion
        if (previousFrame != null) {
            // Compute frame difference
            val frameDiff = Mat()
            Core.absdiff(previousFrame, grayFrame, frameDiff)
            
            // Thresholding to get significant differences
            val threshold = Mat()
            Imgproc.threshold(frameDiff, threshold, 25.0, 255.0, Imgproc.THRESH_BINARY)
            
            // Dilate to enhance moving objects
            val kernel = Imgproc.getStructuringElement(Imgproc.MORPH_ELLIPSE, Size(3.0, 3.0))
            Imgproc.dilate(threshold, threshold, kernel)
            
            // Find contours of moving objects
            val contours = ArrayList<MatOfPoint>()
            Imgproc.findContours(
                threshold.clone(), 
                contours, 
                Mat(), 
                Imgproc.RETR_EXTERNAL, 
                Imgproc.CHAIN_APPROX_SIMPLE
            )
            
            // Filter and analyze potential projectiles
            for (contour in contours) {
                val area = Imgproc.contourArea(contour)
                
                // Filter by size - projectiles are usually small to medium objects
                if (area in 20.0..500.0) {
                    val rect = Imgproc.boundingRect(contour)
                    
                    // Calculate center of the contour
                    val center = Point(rect.x + rect.width / 2, rect.y + rect.height / 2)
                    
                    // Calculate velocity by comparing with previous detections
                    var velocity = Point(0, 0)
                    var matchedPrevious = false
                    
                    for (prevProj in previousProjectiles) {
                        // Calculate Euclidean distance
                        val dx = center.x - prevProj.position.x
                        val dy = center.y - prevProj.position.y
                        val distance = Math.sqrt((dx * dx + dy * dy).toDouble())
                        
                        // If close enough, consider it the same projectile
                        if (distance < 50) {
                            velocity = Point(dx.toInt(), dy.toInt())
                            matchedPrevious = true
                            break
                        }
                    }
                    
                    // Skip objects without velocity (may be UI elements)
                    if (matchedPrevious || previousProjectiles.isEmpty()) {
                        // Additional color-based validation for projectiles
                        // Extract the region for color analysis
                        val roi = Mat(currentFrame, rect)
                        val avgColor = Core.mean(roi)
                        
                        // Brawl Stars projectiles often have saturated colors
                        // Calculate color intensity
                        val intensity = (avgColor.`val`[0] + avgColor.`val`[1] + avgColor.`val`[2]) / 3.0
                        val saturation = calculateSaturation(avgColor)
                        
                        // Higher confidence for bright and saturated objects
                        var confidence = 0.5f
                        if (intensity > 100 && saturation > 0.3) {
                            confidence += 0.3f
                        }
                        
                        // Straight motion patterns increase confidence
                        if (velocity.x != 0 || velocity.y != 0) {
                            confidence += 0.2f
                        }
                        
                        // Create projectile object with calculated properties
                        val projectile = Projectile(
                            position = center,
                            velocity = velocity,
                            size = Math.sqrt(area).toFloat(),
                            confidence = confidence
                        )
                        
                        // Only add high confidence projectiles
                        if (confidence > 0.6f) {
                            detectedProjectiles.add(projectile)
                        }
                    }
                }
            }
            
            // Advanced validation using projectile history
            if (projectileHistory.size > 1) {
                validateProjectilesWithHistory(detectedProjectiles)
            }
        }
        
        // Update history
        previousFrame = grayFrame
        previousProjectiles = detectedProjectiles
        
        // Add to history
        projectileHistory.add(detectedProjectiles)
        if (projectileHistory.size > MAX_HISTORY) {
            projectileHistory.removeFirst()
        }
        
        return detectedProjectiles
    }
    
    private fun calculateSaturation(avgColor: Scalar): Double {
        val r = avgColor.`val`[0]
        val g = avgColor.`val`[1]
        val b = avgColor.`val`[2]
        
        val max = maxOf(r, g, b)
        val min = minOf(r, g, b)
        
        return if (max > 0) (max - min) / max else 0.0
    }
    
    private fun validateProjectilesWithHistory(currentProjectiles: MutableList<Projectile>) {
        // Check if projectiles follow a consistent path
        for (i in currentProjectiles.indices.reversed()) {
            val proj = currentProjectiles[i]
            var consistentMotion = false
            
            // Check each historical frame for this projectile
            val positions = mutableListOf<Point>()
            positions.add(proj.position)
            
            for (pastProjs in projectileHistory.reversed()) {
                var found = false
                
                for (pastProj in pastProjs) {
                    val dx = proj.position.x - pastProj.position.x
                    val dy = proj.position.y - pastProj.position.y
                    val distance = Math.sqrt((dx * dx + dy * dy).toDouble())
                    
                    // If found in history with reasonable distance
                    if (distance < 50 * projectileHistory.indexOf(pastProjs) + 50) {
                        positions.add(pastProj.position)
                        found = true
                        break
                    }
                }
                
                if (!found) break
            }
            
            // If we found this projectile in multiple frames
            if (positions.size >= 3) {
                // Check if motion is somewhat linear
                val linearity = calculateLinearity(positions)
                consistentMotion = linearity > 0.7 // Threshold for linearity
                
                // Boost confidence for consistent projectiles
                if (consistentMotion) {
                    val updatedProjectile = currentProjectiles[i].copy(
                        confidence = minOf(1.0f, currentProjectiles[i].confidence + 0.2f)
                    )
                    currentProjectiles[i] = updatedProjectile
                }
            }
        }
    }
    
    private fun calculateLinearity(points: List<Point>): Double {
        if (points.size < 3) return 1.0
        
        // Simple linearity check: sum of distances from middle points to line formed by endpoints
        val start = points.first()
        val end = points.last()
        
        var totalDeviation = 0.0
        var maxDistance = 0.0
        
        for (i in 1 until points.size - 1) {
            val distToLine = distancePointToLine(points[i], start, end)
            totalDeviation += distToLine
            maxDistance = maxOf(maxDistance, distToLine)
        }
        
        // Calculate linearity score (1.0 = perfect line, less is more deviation)
        val avgDeviation = totalDeviation / (points.size - 2)
        val pointDistance = Math.sqrt(
            Math.pow(end.x - start.x.toDouble(), 2.0) + 
            Math.pow(end.y - start.y.toDouble(), 2.0)
        )
        
        return 1.0 - minOf(1.0, avgDeviation / (pointDistance / 4.0))
    }
    
    private fun distancePointToLine(point: Point, lineStart: Point, lineEnd: Point): Double {
        val x0 = point.x.toDouble()
        val y0 = point.y.toDouble()
        val x1 = lineStart.x.toDouble()
        val y1 = lineStart.y.toDouble()
        val x2 = lineEnd.x.toDouble()
        val y2 = lineEnd.y.toDouble()
        
        // Distance from point to line formula
        return Math.abs((y2 - y1) * x0 - (x2 - x1) * y0 + x2 * y1 - y2 * x1) / 
                Math.sqrt(Math.pow(y2 - y1, 2.0) + Math.pow(x2 - x1, 2.0))
    }
}
```

## DodgeController.kt

```kotlin
package com.autododge.brawlstars

import android.graphics.Point
import java.util.*
import kotlin.math.atan2
import kotlin.math.cos
import kotlin.math.sin
import kotlin.math.sqrt

class DodgeController(private val accessibilityService: AutoDodgeService?) {
    
    companion object {
        private const val DODGE_DISTANCE = 200f // Distance to move during dodge
        private const val PLAYER_RADIUS = 50 // Approximate player collision radius in pixels
        private const val COOLDOWN_TIME = 1000L // Minimum time between dodges in milliseconds
    }
    
    private var lastDodgeTime = 0L
    private val random = Random()
    
    /**
     * Calculate the best dodge direction based on incoming projectiles and execute dodge
     * 
     * @param projectiles List of detected projectiles
     * @param screenWidth Width of screen
     * @param screenHeight Height of screen
     */
    fun calculateAndExecuteDodge(
        projectiles: List<ProjectileDetector.Projectile>,
        screenWidth: Int,
        screenHeight: Int
    ) {
        if (accessibilityService == null || projectiles.isEmpty()) {
            return
        }
        
        // Check cooldown between dodges
        val currentTime = System.currentTimeMillis()
        if (currentTime - lastDodgeTime < COOLDOWN_TIME) {
            return
        }
        
        // Estimate player position (usually in the center of the bottom part of screen)
        val playerX = screenWidth / 2
        val playerY = screenHeight * 3 / 4
        
        // Filter for only threatening projectiles (moving toward player)
        val threateningProjectiles = projectiles.filter { projectile ->
            isProjectileThreateningPlayer(projectile, playerX, playerY)
        }
        
        if (threateningProjectiles.isEmpty()) {
            return
        }
        
        // Find the most urgent threat based on predicted collision time
        val mostUrgentThreat = findMostUrgentThreat(threateningProjectiles, playerX, playerY)
        
        // Calculate best dodge direction
        val dodgeDirection = calculateOptimalDodgeDirection(
            mostUrgentThreat,
            threateningProjectiles,
            playerX,
            playerY,
            screenWidth,
            screenHeight
        )
        
        // Execute the dodge
        accessibilityService.executeDodge(dodgeDirection, DODGE_DISTANCE, screenWidth, screenHeight)
        lastDodgeTime = currentTime
    }
    
    /**
     * Determine if a projectile is moving toward the player
     */
    private fun isProjectileThreateningPlayer(
        projectile: ProjectileDetector.Projectile,
        playerX: Int,
        playerY: Int
    ): Boolean {
        // Get projectile position and velocity
        val projX = projectile.position.x
        val projY = projectile.position.y
        val velocityX = projectile.velocity.x
        val velocityY = projectile.velocity.y
        
        // If velocity is 0, it's not moving
        if (velocityX == 0 && velocityY == 0) {
            return false
        }
        
        // Calculate direction from projectile to player
        val dirToPlayerX = playerX - projX
        val dirToPlayerY = playerY - projY
        
        // Calculate dot product of velocity and direction to player
        val dotProduct = velocityX * dirToPlayerX + velocityY * dirToPlayerY
        
        // If dot product is positive, projectile is moving toward player
        return dotProduct > 0
    }
    
    /**
     * Find the most urgent threat based on predicted collision time
     */
    private fun findMostUrgentThreat(
        projectiles: List<ProjectileDetector.Projectile>,
        playerX: Int,
        playerY: Int
    ): ProjectileDetector.Projectile {
        return projectiles.minByOrNull { projectile ->
            val projX = projectile.position.x
            val projY = projectile.position.y
            val velocityX = projectile.velocity.x
            val velocityY = projectile.velocity.y
            
            // Calculate distance to player
            val distX = playerX - projX
            val distY = playerY - projY
            val distance = sqrt((distX * distX + distY * distY).toDouble()).toFloat()
            
            // Calculate speed of projectile
            val speed = sqrt((velocityX * velocityX + velocityY * velocityY).toDouble()).toFloat()
            
            // Time to collision = distance / speed
            if (speed > 0) distance / speed else Float.MAX_VALUE
        } ?: projectiles.first()
    }
    
    /**
     * Calculate the optimal dodge direction to avoid projectiles
     */
    private fun calculateOptimalDodgeDirection(
        urgentThreat: ProjectileDetector.Projectile,
        allThreats: List<ProjectileDetector.Projectile>,
        playerX: Int,
        playerY: Int,
        screenWidth: Int,
        screenHeight: Int
    ): Float {
        // Get threat position and velocity
        val threatX = urgentThreat.position.x
        val threatY = urgentThreat.position.y
        val threatVelX = urgentThreat.velocity.x
        val threatVelY = urgentThreat.velocity.y
        
        // Calculate angle of incoming threat (perpendicular is best dodge direction)
        val threatAngle = atan2(threatVelY.toDouble(), threatVelX.toDouble())
        
        // Try both perpendicular directions (90 degrees to left and right)
        val leftDodgeAngle = threatAngle + Math.PI / 2
        val rightDodgeAngle = threatAngle - Math.PI / 2
        
        // Check which direction is safer by simulating both dodges
        val leftDodgeScore = calculateDodgeSafetyScore(
            leftDodgeAngle, 
            allThreats, 
            playerX, 
            playerY, 
            screenWidth, 
            screenHeight
        )
        
        val rightDodgeScore = calculateDodgeSafetyScore(
            rightDodgeAngle, 
            allThreats, 
            playerX, 
            playerY, 
            screenWidth, 
            screenHeight
        )
        
        // Choose the direction with higher safety score
        val bestAngle = if (leftDodgeScore >= rightDodgeScore) leftDodgeAngle else rightDodgeAngle
        
        // Convert to degrees and normalize to 0-360 range
        val degrees = Math.toDegrees(bestAngle)
        return ((degrees + 360) % 360).toFloat()
    }
    
    /**
     * Calculate a safety score for a potential dodge direction
     * Higher score means safer dodge
     */
    private fun calculateDodgeSafetyScore(
        dodgeAngle: Double,
        threats: List<ProjectileDetector.Projectile>,
        playerX: Int,
        playerY: Int,
        screenWidth: Int,
        screenHeight: Int
    ): Double {
        // Calculate position after dodge
        val dodgeX = playerX + DODGE_DISTANCE * cos(dodgeAngle).toFloat()
        val dodgeY = playerY - DODGE_DISTANCE * sin(dodgeAngle).toFloat()  // Negative because Y increases downward
        
        // Check if position would be off screen
        if (dodgeX < PLAYER_RADIUS || dodgeX > screenWidth - PLAYER_RADIUS ||
            dodgeY < PLAYER_RADIUS || dodgeY > screenHeight - PLAYER_RADIUS) {
            return 0.0  // Avoid going off screen
        }
        
        var safetyScore = 100.0
        
        // Check danger from each threat at the new position
        for (threat in threats) {
            val threatX = threat.position.x
            val threatY = threat.position.y
            val threatVelX = threat.velocity.x
            val threatVelY = threat.velocity.y
            
            // Skip threats with no velocity
            if (threatVelX == 0 && threatVelY == 0) continue
            
            // Predict threat position after dodge completion
            val timeForDodge = 0.2  // 200ms dodge time
            val predictedThreatX = threatX + threatVelX * timeForDodge
            val predictedThreatY = threatY + threatVelY * timeForDodge
            
            // Calculate distance between dodge position and predicted threat
            val distX = dodgeX - predictedThreatX
            val distY = dodgeY - predictedThreatY
            val distance = sqrt((distX * distX + distY * distY).toDouble())
            
            // Reduce score based on how close threat will be
            if (distance < PLAYER_RADIUS + threat.size) {
                // Would be a collision
                safetyScore -= 50
            } else if (distance < 3 * PLAYER_RADIUS) {
                // Close call
                safetyScore -= 30 * (3 * PLAYER_RADIUS - distance) / (2 * PLAYER_RADIUS)
            }
        }
        
        return maxOf(0.0, safetyScore)
    }
}
```

## Utils files

### GestureUtils.kt

```kotlin
package com.autododge.brawlstars.utils

import android.accessibilityservice.GestureDescription
import android.graphics.Path
import android.os.Build
import androidx.annotation.RequiresApi

/**
 * Utility class for creating gesture descriptions for the accessibility service
 */
object GestureUtils {

    /**
     * Creates a GestureDescription for the given path
     *
     * @param path The gesture path
     * @param startTime The start time of the gesture in ms
     * @param duration The duration of the gesture in ms
     * @return A GestureDescription that can be dispatched by an accessibility service
     */
    fun createGesture(path: Path, startTime: Long, duration: Long): GestureDescription {
        val builder = GestureDescription.Builder()
        val strokeDescription = GestureDescription.StrokeDescription(
            path,
            startTime,
            duration
        )
        
        return builder.addStroke(strokeDescription).build()
    }
    
    /**
     * Creates a swipe gesture
     *
     * @param startX Starting X coordinate
     * @param startY Starting Y coordinate
     * @param endX Ending X coordinate
     * @param endY Ending Y coordinate
     * @param duration Duration of the swipe in ms
     * @return A GestureDescription for the swipe action
     */
    fun createSwipeGesture(
        startX: Float,
        startY: Float,
        endX: Float,
        endY: Float,
        duration: Long
    ): GestureDescription {
        val path = Path()
        path.moveTo(startX, startY)
        path.lineTo(endX, endY)
        
        return createGesture(path, 0, duration)
    }
    
    /**
     * Creates a tap gesture
     *
     * @param x X coordinate to tap
     * @param y Y coordinate to tap
     * @return A GestureDescription for the tap action
     */
    fun createTapGesture(x: Float, y: Float): GestureDescription {
        val path = Path()
        path.moveTo(x, y)
        path.lineTo(x, y)
        
        return createGesture(path, 0, 100)
    }
    
    /**
     * Creates a double tap gesture
     *
     * @param x X coordinate to tap
     * @param y Y coordinate to tap
     * @return A GestureDescription for the double tap action
     */
    fun createDoubleTapGesture(x: Float, y: Float): GestureDescription {
        val path = Path()
        path.moveTo(x, y)
        path.lineTo(x, y)
        
        val builder = GestureDescription.Builder()
        val firstTap = GestureDescription.StrokeDescription(path, 0, 100)
        val secondTap = GestureDescription.StrokeDescription(path, 200, 100)
        
        return builder
            .addStroke(firstTap)
            .addStroke(secondTap)
            .build()
    }
    
    /**
     * Creates a circular gesture
     *
     * @param centerX Center X coordinate of the circle
     * @param centerY Center Y coordinate of the circle
     * @param radius Radius of the circle in pixels
     * @param duration Duration of the circular motion in ms
     * @return A GestureDescription for the circular motion
     */
    fun createCircularGesture(
        centerX: Float,
        centerY: Float,
        radius: Float,
        duration: Long
    ): GestureDescription {
        val path = Path()
        path.addCircle(centerX, centerY, radius, Path.Direction.CW)
        
        return createGesture(path, 0, duration)
    }
}
```

### ImageProcessingUtils.kt

```kotlin
package com.autododge.brawlstars.utils

import android.graphics.Bitmap
import android.graphics.Canvas
import android.graphics.Color
import android.graphics.Paint
import android.graphics.Point
import com.autododge.brawlstars.ProjectileDetector

/**
 * Utility functions for image processing and visualization
 */
object ImageProcessingUtils {
    
    /**
     * Draws detected projectiles on a bitmap for visualization purposes
     * 
     * @param bitmap Original bitmap to draw onto
     * @param projectiles List of detected projectiles
     * @return A new bitmap with projectiles visualization
     */
    fun drawProjectilesOnBitmap(bitmap: Bitmap, projectiles: List<ProjectileDetector.Projectile>): Bitmap {
        val visualizationBitmap = bitmap.copy(Bitmap.Config.ARGB_8888, true)
        val canvas = Canvas(visualizationBitmap)
        val paint = Paint().apply {
            style = Paint.Style.STROKE
            strokeWidth = 3f
        }
        
        val velocityPaint = Paint().apply {
            color = Color.RED
            strokeWidth = 2f
        }
        
        for (projectile in projectiles) {
            // Color based on confidence
            when {
                projectile.confidence > 0.8f -> paint.color = Color.RED     // High confidence
                projectile.confidence > 0.7f -> paint.color = Color.YELLOW  // Medium confidence
                else -> paint.color = Color.GREEN                           // Low confidence
            }
            
            // Draw circle at projectile position
            canvas.drawCircle(
                projectile.position.x.toFloat(),
                projectile.position.y.toFloat(),
                projectile.size,
                paint
            )
            
            // Draw velocity vector
            if (projectile.velocity.x != 0 || projectile.velocity.y != 0) {
                val startX = projectile.position.x.toFloat()
                val startY = projectile.position.y.toFloat()
                val endX = startX + projectile.velocity.x * 3  // Scale for visibility
                val endY = startY + projectile.velocity.y * 3
                
                canvas.drawLine(startX, startY, endX, endY, velocityPaint)
            }
            
            // Draw confidence text
            val textPaint = Paint().apply {
                color = Color.WHITE
                textSize = 20f
            }
            canvas.drawText(
                String.format("%.2f", projectile.confidence),
                projectile.position.x.toFloat(),
                projectile.position.y.toFloat() - projectile.size - 5,
                textPaint
            )
        }
        
        return visualizationBitmap
    }
    
    /**
     * Converts YUV format (from camera) to Bitmap
     */
    fun convertYUVToBitmap(data: ByteArray, width: Int, height: Int): Bitmap {
        val bitmap = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888)
        val rgba = IntArray(width * height)
        
        for (i in 0 until height) {
            for (j in 0 until width) {
                val yIndex = i * width + j
                val y = data[yIndex].toInt() and 0xFF
                
                // YUV to RGB conversion (simplified)
                rgba[yIndex] = Color.rgb(y, y, y)
            }
        }
        
        bitmap.setPixels(rgba, 0, width, 0, 0, width, height)
        return bitmap
    }
    
    /**
     * Resizes bitmap for faster processing
     */
    fun resizeBitmap(bitmap: Bitmap, targetWidth: Int, targetHeight: Int): Bitmap {
        return Bitmap.createScaledBitmap(bitmap, targetWidth, targetHeight, true)
    }
    
    /**
     * Calculate color distance between two RGB values
     */
    fun colorDistance(rgb1: Int, rgb2: Int): Double {
        val r1 = Color.red(rgb1)
        val g1 = Color.green(rgb1)
        val b1 = Color.blue(rgb1)
        
        val r2 = Color.red(rgb2)
        val g2 = Color.green(rgb2)
        val b2 = Color.blue(rgb2)
        
        val dr = (r1 - r2).toDouble()
        val dg = (g1 - g2).toDouble()
        val db = (b1 - b2).toDouble()
        
        return Math.sqrt(dr * dr + dg * dg + db * db)
    }
    
    /**
     * Calculate average color in a region
     */
    fun averageColor(bitmap: Bitmap, x: Int, y: Int, size: Int): Int {
        var r = 0
        var g = 0
        var b = 0
        var pixelCount = 0
        
        val startX = Math.max(0, x - size / 2)
        val startY = Math.max(0, y - size / 2)
        val endX = Math.min(bitmap.width - 1, x + size / 2)
        val endY = Math.min(bitmap.height - 1, y + size / 2)
        
        for (i in startY..endY) {
            for (j in startX..endX) {
                val pixel = bitmap.getPixel(j, i)
                r += Color.red(pixel)
                g += Color.green(pixel)
                b += Color.blue(pixel)
                pixelCount++
            }
        }
        
        if (pixelCount == 0) return Color.BLACK
        
        r /= pixelCount
        g /= pixelCount
        b /= pixelCount
        
        return Color.rgb(r, g, b)
    }
}
```

### ScreenCaptureManager.kt

```kotlin
package com.autododge.brawlstars.utils

import android.content.Context
import android.graphics.Bitmap
import android.graphics.PixelFormat
import android.hardware.display.DisplayManager
import android.hardware.display.VirtualDisplay
import android.media.Image
import android.media.ImageReader
import android.media.projection.MediaProjection
import android.os.Handler
import android.os.Looper
import android.util.DisplayMetrics
import android.view.Surface
import android.view.WindowManager

class ScreenCaptureManager(private val context: Context, private val mediaProjection: MediaProjection) {
    
    companion object {
        private const val VIRTUAL_DISPLAY_NAME = "auto_dodge_screen_capture"
        private const val IMAGE_QUALITY = 3 // Reduce quality for better performance (1-10)
    }
    
    private var virtualDisplay: VirtualDisplay? = null
    private var imageReader: ImageReader? = null
    private val handler = Handler(Looper.getMainLooper())
    private var screenWidth: Int = 0
    private var screenHeight: Int = 0
    private var screenDensity: Int = 0
    
    init {
        setupScreenMetrics()
        setupImageReader()
        createVirtualDisplay()
    }
    
    private fun setupScreenMetrics() {
        val windowManager = context.getSystemService(Context.WINDOW_SERVICE) as WindowManager
        val metrics = DisplayMetrics()
        windowManager.defaultDisplay.getMetrics(metrics)
        
        screenWidth = metrics.widthPixels / IMAGE_QUALITY
        screenHeight = metrics.heightPixels / IMAGE_QUALITY
        screenDensity = metrics.densityDpi
    }
    
    private fun setupImageReader() {
        imageReader = ImageReader.newInstance(
            screenWidth, 
            screenHeight, 
            PixelFormat.RGBA_8888, 
            2 // Max images
        )
    }
    
    private fun createVirtualDisplay() {
        virtualDisplay = mediaProjection.createVirtualDisplay(
            VIRTUAL_DISPLAY_NAME,
            screenWidth,
            screenHeight,
            screenDensity,
            DisplayManager.VIRTUAL_DISPLAY_FLAG_AUTO_MIRROR,
            imageReader?.surface,
            null,
            handler
        )
    }
    
    /**
     * Captures the current screen and passes the bitmap to the callback
     */
    fun captureScreen(callback: (Bitmap) -> Unit) {
        try {
            imageReader?.let { reader ->
                val image = reader.acquireLatestImage()
                
                if (image != null) {
                    val bitmap = imageToBitmap(image)
                    callback(bitmap)
                    image.close()
                }
            }
        } catch (e: Exception) {
            e.printStackTrace()
        }
    }
    
    /**
     * Converts an Image to a Bitmap
     */
    private fun imageToBitmap(image: Image): Bitmap {
        val width = image.width
        val height = image.height
        
        val planes = image.planes
        val buffer = planes[0].buffer
        val pixelStride = planes[0].pixelStride
        val rowStride = planes[0].rowStride
        val rowPadding = rowStride - pixelStride * width
        
        val bitmap = Bitmap.createBitmap(
            width + rowPadding / pixelStride, 
            height, 
            Bitmap.Config.ARGB_8888
        )
        bitmap.copyPixelsFromBuffer(buffer)
        
        return Bitmap.createBitmap(bitmap, 0, 0, width, height)
    }
    
    /**
     * Releases resources used by screen capture
     */
    fun release() {
        virtualDisplay?.release()
        mediaProjection.stop()
        imageReader?.close()
    }
    
    /**
     * Returns current screen width
     */
    fun getScreenWidth(): Int {
        return screenWidth * IMAGE_QUALITY
    }
    
    /**
     * Returns current screen height
     */
    fun getScreenHeight(): Int {
        return screenHeight * IMAGE_QUALITY
    }
}
```

## XML Layouts

### activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp"
    android:background="#121212"
    tools:context=".MainActivity">

    <ImageView
        android:id="@+id/appLogo"
        android:layout_width="120dp"
        android:layout_height="120dp"
        android:layout_marginTop="32dp"
        android:src="@android:drawable/ic_menu_compass"
        android:tint="#FF9800"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <TextView
        android:id="@+id/appTitle"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:text="Brawl Stars Auto Dodge"
        android:textColor="#FF9800"
        android:textSize="24sp"
        android:textStyle="bold"
        app:layout_constraintTop_toBottomOf="@id/appLogo"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />
    
    <TextView
        android:id="@+id/appDescription"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="24dp"
        android:text="This app helps you automatically dodge projectiles in Brawl Stars."
        android:textColor="#FFFFFF"
        android:textSize="16sp"
        android:textAlignment="center"
        app:layout_constraintTop_toBottomOf="@id/appTitle"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <TextView
        android:id="@+id/permissionTitle"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="36dp"
        android:text="Permissions Required:"
        android:textColor="#FF9800"
        android:textSize="18sp"
        app:layout_constraintTop_toBottomOf="@id/appDescription"
        app:layout_constraintStart_toStartOf="parent" />

    <TextView
        android:id="@+id/permissionStatusText"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="12dp"
        android:text="Overlay permission: ❌\nAccessibility permission: ❌"
        android:textColor="#FFFFFF"
        android:textSize="16sp"
        app:layout_constraintTop_toBottomOf="@id/permissionTitle"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <TextView
        android:id="@+id/instructionText"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="36dp"
        android:text="To use Auto Dodge:\n1. Grant all required permissions\n2. Start the service\n3. Open Brawl Stars\n4. Toggle auto dodge from the floating button"
        android:textColor="#CCCCCC"
        android:textSize="14sp"
        app:layout_constraintTop_toBottomOf="@id/permissionStatusText"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <Button
        android:id="@+id/startServiceButton"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="36dp"
        android:text="START AUTO DODGE"
        android:textSize="16sp"
        android:paddingVertical="12dp"
        android:backgroundTint="#FF9800"
        android:enabled="false"
        app:layout_constraintTop_toBottomOf="@id/instructionText"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <TextView
        android:id="@+id/disclaimerText"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="36dp"
        android:text="Note: Use at your own risk. This app is not affiliated with Supercell or Brawl Stars."
        android:textColor="#999999"
        android:textSize="12sp"
        android:textStyle="italic"
        android:textAlignment="center"
        app:layout_constraintTop_toBottomOf="@id/startServiceButton"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

### overlay_controls.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    app:cardBackgroundColor="#90000000"
    app:cardCornerRadius="16dp"
    app:cardElevation="4dp">

    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="8dp">

        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:gravity="center_vertical">

            <ImageView
                android:layout_width="24dp"
                android:layout_height="24dp"
                android:src="@android:drawable/ic_menu_compass"
                android:tint="#FF9800" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Auto Dodge"
                android:textColor="#FFFFFF"
                android:textSize="16sp"
                android:layout_marginStart="8dp" />

            <Switch
                android:id="@+id/autoDodgeSwitch"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_marginStart="8dp" />

        </LinearLayout>

        <Button
            android:id="@+id/closeButton"
            android:layout_width="match_parent"
            android:layout_height="36dp"
            android:layout_marginTop="8dp"
            android:background="@android:color/transparent"
            android:text="Close"
            android:textColor="#FF9800"
            android:textSize="14sp" />

    </LinearLayout>
</androidx.cardview.widget.CardView>
```

## AndroidManifest.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.autododge.brawlstars">

    <!-- Permissions -->
    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
    <uses-permission android:name="android.permission.BIND_ACCESSIBILITY_SERVICE" />

    <application
        android:allowBackup="true"
        android:icon="@android:drawable/ic_menu_compass"
        android:label="Brawl Stars Auto Dodge"
        android:roundIcon="@android:drawable/ic_menu_compass"
        android:supportsRtl="true"
        android:theme="@style/Theme.AppCompat">

        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <service
            android:name=".OverlayService"
            android:enabled="true"
            android:exported="false"
            android:foregroundServiceType="mediaProjection" />

        <service
            android:name=".AutoDodgeService"
            android:enabled="true"
            android:exported="false"
            android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE">
            <intent-filter>
                <action android:name="android.accessibilityservice.AccessibilityService" />
            </intent-filter>

            <meta-data
                android:name="android.accessibilityservice"
                android:resource="@xml/accessibility_service_config" />
        </service>

    </application>

</manifest>
```

## accessibility_service_config.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<accessibility-service
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:description="@string/accessibility_service_description"
    android:accessibilityEventTypes="typeAllMask"
    android:accessibilityFlags="flagDefault"
    android:accessibilityFeedbackType="feedbackGeneric"
    android:notificationTimeout="100"
    android:canPerformGestures="true"
    android:canRetrieveWindowContent="true" />
```

## Gradle Build Files

### app/build.gradle

```gradle
plugins {
    id 'com.android.application'
    id 'kotlin-android'
}

android {
    compileSdkVersion 33
    defaultConfig {
        applicationId "com.autododge.brawlstars"
        minSdkVersion 24
        targetSdkVersion 33
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    kotlinOptions {
        jvmTarget = '1.8'
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.7.10"
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'androidx.core:core-ktx:1.9.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
    implementation 'androidx.cardview:cardview:1.0.0'
    implementation 'com.google.android.material:material:1.9.0'
    
    // OpenCV for Android
    implementation 'org.opencv:opencv-android:4.6.0'
    
    testImplementation 'junit:junit:4.13.2'
    androidTestImplementation 'androidx.test.ext:junit:1.1.5'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.5.1'
}
```

### build.gradle (Project level)

```gradle
// Top-level build file where you can add configuration options common to all sub-projects/modules.
buildscript {
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:7.3.1'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:1.7.10"
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        google()
        mavenCentral()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

## Building Instructions

For detailed instructions on building this app, please refer to:
- [BUILDING_GUIDE.md](BUILDING_GUIDE.md) - How to build on a computer with Android Studio
- [MOBILE_BUILDING_GUIDE.md](MOBILE_BUILDING_GUIDE.md) - How to attempt building on your Android phone