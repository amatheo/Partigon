# Partigon Tutorial: Creating a Simple Linear Particle Animation

This tutorial will guide you through creating your first particle animation using Partigon. We'll create a simple linear particle animation that moves a particle from one point to another.

## Prerequisites

- A Minecraft server running Paper or Spigot
- Basic knowledge of Kotlin and Bukkit/Paper API
- Partigon library added to your project dependencies

## Core Concepts

Before we start, let's understand the key concepts in Partigon:

### 1. Animation
An `Animation` is a single particle animation that displays particles every frame. It's controlled by envelopes that define how particle properties change over time.

### 2. Envelopes
Envelopes control how particle properties (position, offset, count, etc.) change over time. They interpolate values between frames.

- **LinearEnvelope**: Interpolates linearly between two values
- **ConstantEnvelope**: Holds a constant value
- **BasicEnvelope**: Custom envelope with mathematical expressions

### 3. Loops
Loops control how envelopes behave over time:

- **SingleIteration**: Plays once and stops
- **RepeatLoop**: Repeats indefinitely
- **BounceLoop**: Bounces back and forth
- **ReverseLoop**: Plays in reverse
- **ContinueLoop**: Continues with the last value

### 4. PartigonLocation
Defines where the animation originates. Can be:

- **ConstantLocation**: A fixed location
- **EntityLocation**: Follows an entity

## Creating Your First Linear Animation

Let's create a simple animation that moves a particle from point A to point B in a straight line.

### Step 1: Set Up the Basic Animation

```kotlin
import kotlinx.coroutines.CoroutineScope
import org.bukkit.Location
import org.bukkit.Particle
import xyz.gameoholic.partigon.particle.Animation
import xyz.gameoholic.partigon.particle.envelope.LinearEnvelope
import xyz.gameoholic.partigon.particle.location.ConstantLocation
import xyz.gameoholic.partigon.particle.loop.SingleIteration
import xyz.gameoholic.partigon.util.envelope
import kotlin.time.Duration.Companion.seconds

// Create the animation using the builder pattern
val animation = Animation.singleAnimation {
    // Set the origin location (starting point)
    originLocation = ConstantLocation(
        Location(world, 0.0, 64.0, 0.0)
    )
    
    // Set the particle type
    particleType = Particle.END_ROD
    
    // Set the particle count (1 particle per frame)
    count = 1.envelope
}
```

### Step 2: Add Linear Movement

Now let's make the particle move from (0, 64, 0) to (10, 64, 0) over 3 seconds (60 ticks):

```kotlin
val animation = Animation.singleAnimation {
    originLocation = ConstantLocation(
        Location(world, 0.0, 64.0, 0.0)
    )
    particleType = Particle.END_ROD
    count = 1.envelope
    
    // Define movement duration (3 seconds = 60 ticks)
    val duration = 60
    val loop = SingleIteration(duration)
    
    // Linear movement on X axis: from 0 to 10
    positionX = LinearEnvelope(
        value1 = 0.0.envelope,      // Start value
        value2 = 10.0.envelope,     // End value
        loop = loop
    )
    
    // Y and Z remain at 0 (relative to origin)
    positionY = 0.0.envelope
    positionZ = 0.0.envelope
    
    // Set maximum duration
    maximumDuration = 3.seconds
}
```

### Step 3: Start the Animation

To play the animation, you need to provide a CoroutineScope:

```kotlin
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.Dispatchers

// Start the animation
val scope = CoroutineScope(Dispatchers.Default)
animation.start(scope)
```

### Step 4: Control the Animation

You can control the animation with these methods:

```kotlin
// Stop the animation
animation.stop()

// Resume from where it stopped
animation.resume()

// Resume with a different scope
animation.resume(newScope)
```

## Complete Example

Here's a complete working example that creates a particle moving in a linear path:

```kotlin
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.Dispatchers
import org.bukkit.Location
import org.bukkit.Particle
import org.bukkit.World
import xyz.gameoholic.partigon.particle.Animation
import xyz.gameoholic.partigon.particle.envelope.LinearEnvelope
import xyz.gameoholic.partigon.particle.location.ConstantLocation
import xyz.gameoholic.partigon.particle.loop.SingleIteration
import xyz.gameoholic.partigon.util.envelope
import kotlin.time.Duration.Companion.seconds

fun createLinearParticleAnimation(world: World): Animation {
    return Animation.singleAnimation {
        // Origin point
        originLocation = ConstantLocation(
            Location(world, 0.0, 64.0, 0.0)
        )
        
        // Use glowing particles
        particleType = Particle.END_ROD
        
        // One particle per frame
        count = 1.envelope
        
        // Animation lasts 3 seconds (60 ticks at 20 tps)
        val animationDuration = 60
        val loop = SingleIteration(animationDuration)
        
        // Move from X=0 to X=10
        positionX = LinearEnvelope(
            value1 = 0.0.envelope,
            value2 = 10.0.envelope,
            loop = loop
        )
        
        // Move from Y=0 to Y=5 (creates an arc)
        positionY = LinearEnvelope(
            value1 = 0.0.envelope,
            value2 = 5.0.envelope,
            loop = loop
        )
        
        // Stay at Z=0
        positionZ = 0.0.envelope
        
        // Set offsets to 0 (no random spread)
        offsetX = 0.0.envelope
        offsetY = 0.0.envelope
        offsetZ = 0.0.envelope
        
        // Set maximum duration
        maximumDuration = 3.seconds
    }
}

// Usage in a command or event handler:
fun playAnimation(world: World) {
    val animation = createLinearParticleAnimation(world)
    val scope = CoroutineScope(Dispatchers.Default)
    animation.start(scope)
}
```

## Advanced Tips

### Multiple Particles
To create a trail effect, increase the particle count:

```kotlin
count = 5.envelope  // Spawns 5 particles per frame
```

### Adding Particle Offset
Add randomness to particle position using offsets:

```kotlin
offsetX = 0.5.envelope  // Random spread of 0.5 blocks in X
offsetY = 0.5.envelope
offsetZ = 0.5.envelope
```

### Colored Particles (Dust)
For colored particles, use DustOptions:

```kotlin
import org.bukkit.Color
import org.bukkit.Particle.DustOptions

particleType = Particle.DUST
dustOptions = DustOptions(Color.RED, 1.0f)  // Red dust, size 1.0
```

### Repeating Animations
To make the animation loop, use RepeatLoop:

```kotlin
import xyz.gameoholic.partigon.particle.loop.RepeatLoop

val loop = RepeatLoop(60)  // Repeats every 60 ticks
```

### Animation Interval
Control how often the animation updates:

```kotlin
animationInterval = 2  // Update every 2 ticks instead of every tick
```

## Common Use Cases

### 1. Straight Line (Projectile Trail)
```kotlin
positionX = LinearEnvelope(0.0.envelope, 20.0.envelope, SingleIteration(100))
positionY = 0.0.envelope
positionZ = 0.0.envelope
```

### 2. Diagonal Movement
```kotlin
positionX = LinearEnvelope(0.0.envelope, 10.0.envelope, SingleIteration(60))
positionY = LinearEnvelope(0.0.envelope, 10.0.envelope, SingleIteration(60))
positionZ = 0.0.envelope
```

### 3. Rising Particle
```kotlin
positionX = 0.0.envelope
positionY = LinearEnvelope(0.0.envelope, 5.0.envelope, SingleIteration(40))
positionZ = 0.0.envelope
```

## Troubleshooting

**Particles not appearing?**
- Check that the world is correct
- Ensure players are within render distance
- Verify the particle type is valid
- Make sure the animation is started with a valid scope

**Animation stops immediately?**
- Check that maximumDuration is set correctly
- Verify the loop duration matches your expected animation length
- Ensure the scope hasn't been cancelled

**Particles spawn at wrong location?**
- Remember that position envelopes are **relative** to the origin location
- The origin location is your base point, and position envelopes add to it

## Next Steps

Now that you understand basic linear animations, you can explore:
- **MultiAnimation**: Combine multiple animations to play in sync
- **EnvelopeGroups**: Apply transformations to groups of envelopes
- **Rotation**: Add rotation to your particles
- **Trigonometric Envelopes**: Create circular or wave patterns
- **Custom Envelopes**: Use mathematical expressions for complex movements

For more advanced examples, check the source code in the `particle` package.
