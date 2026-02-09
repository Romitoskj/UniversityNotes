The track-following algorithm in the onTrek wear app guides users along a predefined GPX track. Here's how it works:  
  
## Overview  
  
The algorithm determines **which track point the user should navigate to next** and whether they are **on-track or off-track**, then provides directional guidance via an arrow.  
  
---  
  
## Key Components  
  
### 1. **Track Points**  
The GPX file is parsed into a list of `TrackPoint` objects, each containing:  
- Latitude/Longitude coordinates  
- Distance to the previous point  
- Cumulative distance traveled  
- An index in the sequence  
  
### 2. **Core Thresholds**  
| Constant | Value | Purpose |  
|----------|-------|---------|  
| `trackPointThreshold` | 15m | Point is considered "reached" when user is within this distance |  
| `notificationTrackDistanceThreshold` | 25m | Distance beyond which user is considered off-track |  
| `degreesThreshold` | 5° | Minimum rotation to update the direction arrow |  
  
---  
  
## Main Algorithm: `findNextTrackPoint()`  
  
This function determines which track point the user should navigate toward.  
  
### Step 1: Find the Probable Next Point  
- **First call (no previous context):** Finds the nearest point by checking distance to all track points using `getNearestPoints()`. If the nearest point is close to the end, it looks for a better starting point near the beginning (handles circular tracks).  
- **Subsequent calls:** Uses `extractNearestPoint()` which searches from the last known position, optimizing performance.  
  
### Step 2: Skip Points Already Reached  
If the user is within `trackPointThreshold` (15m) of the probable point:  
- Advances to the next point in the sequence  
- Repeats until finding a point beyond the threshold  
  
### Step 3: Decide Between Current Point or Next Point  
Uses a geometric comparison (referenced in [GitHub issue #40](https://github.com/onTrek/android/issues/40)):  
  
```  
Point W = current probable point  
Point W-1 = previous point  Point W+1 = next point  
  
A = distance from user to W  
B = distance from user to W-1  C = distance from user to W+1  
X = distance between W-1 and W (track segment)  
Y = distance between W and W+1 (track segment)  
  
offset1 = (A + B) - X  offset2 = (A + C) - Y  
```  
  
- If `offset2 < offset1` AND `offset2 < 25m` → User should navigate to **W+1** (they're closer to the next segment)  
- Otherwise → User should navigate to **W** (the current point)  
  
---  
  
## Off-Track Detection: `computeDistanceFromTrack()`  
  
Calculates how far the user has deviated from the track:  
  
```kotlin  
distanceFromTrack = distanceToNextPoint + distanceToPreviousPoint - segmentLength  
```  
  
This measures the **perpendicular deviation** from the track segment. If this value exceeds `notificationTrackDistanceThreshold` (25m), the user is considered off-track.  
  
---  
  
## Direction Calculation: `elaborateDirection()`  
  
Uses spherical trigonometry (bearing calculation) to determine the arrow direction:  
  
1. Calculates the **bearing** from user's position to the next track point using:  
   ```kotlin  
   targetBearing = atan2(  
       sin(deltaLon) * cos(lat2),       cos(lat1) * sin(lat2) - sin(lat1) * cos(lat2) * cos(deltaLon)   )  
   ```  
2. Combines with **compass direction** to show a relative arrow:  
   ```kotlin  
   arrowAngle = (compassDirection - targetBearing + 360) % 360  
   ```  
3. Only updates if the change exceeds `degreesThreshold` (5°) to prevent jitter.  
  
---  
  
## Progress Tracking  
  
```kotlin  
distanceTraveled = nextTrackPoint.totalDistanceTraveled - distanceToNextPoint  
progress = distanceTraveled / totalTrackLength  
remainingDistance = totalLength - distanceTraveled  
```  
  
Progress is only computed when the user is on-track.  
  
---  
  
## State Machine  
  
```  
┌─────────────────────────────────────────────────────────────────┐  
│                         INITIAL STATE                           │  
│                    (hasBeenNearTheTrack = null)                 │  
└───────────────────────────┬─────────────────────────────────────┘  
                            │            ┌───────────────┴───────────────┐            ▼                               ▼    distance < 25m                   distance >= 25m            │                               │            ▼                               ▼┌───────────────────────┐       ┌───────────────────────┐  
│  ON TRACK             │       │  FAR FROM TRACK       │  
│  (hasBeenNear = true) │       │  (hasBeenNear = false)│  
│  Progress computed    │       │  Show distance to     │  
│  Arrow points to next │       │  nearest point        │  
└───────────┬───────────┘       └───────────┬───────────┘  
            │                               │            ▼                               ▼    distance > 25m                   distance < 25m            │                               │            ▼                               ▼┌───────────────────────┐       ┌───────────────────────┐  
│  OFF TRACK            │       │  JOINS TRACK          │  
│  Notification shown   │       │  (transitions to      │  
│  probablePoint frozen │◄──────│   ON TRACK state)     │  
│  Arrow → nearest point│       └───────────────────────┘  
└───────────────────────┘  
```  
  
---  
  
## Optimization: `extractNearestPoint()`  
  
Instead of checking all track points every update, this function:  
  
1. Starts searching from the last known position  
2. Wraps around the track (handles loops)  
3. Stops early once it finds a point below the threshold and distances start increasing  
4. Avoids jumping to points near the start if the user hasn't been near the track yet (prevents false matches on circular tracks)  
  
---  
  
## Summary  
  
The algorithm efficiently tracks user progress by:  
1. **Finding the nearest segment** on the track  
2. **Choosing the best next waypoint** using geometric analysis  
3. **Detecting off-track deviation** using segment distance calculation  
4. **Providing real-time directional guidance** via bearing calculation  
5. **Computing progress** as a percentage of total track length