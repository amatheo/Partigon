# Partigon Tutorial: Creating a Simple Linear Particle Animation (Java)

This tutorial will guide you through creating your first particle animation using Partigon from Java. We'll create a simple linear particle animation that moves a particle from one point to another.

> **Note**: This is a fork of the original Partigon library. The original documentation is available at [partigon.gameoholic.xyz](https://partigon.gameoholic.xyz/), though class names have changed (e.g., `PartigonParticle` → `PartigonAnimation`).

## Prerequisites

- A Minecraft server running Paper or Spigot
- Basic knowledge of Java and Bukkit/Paper API
- Partigon library added to your project dependencies
- Kotlin standard library and Kotlinx Coroutines in your classpath (required by Partigon)

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

```java
import kotlinx.coroutines.CoroutineScope;
import kotlinx.coroutines.Dispatchers;
import org.bukkit.Location;
import org.bukkit.Particle;
import org.bukkit.World;
import xyz.gameoholic.partigon.particle.Animation;
import xyz.gameoholic.partigon.particle.envelope.ConstantEnvelope;
import xyz.gameoholic.partigon.particle.location.ConstantLocation;
import xyz.gameoholic.partigon.particle.envelope.PropertyType;

// Create the animation using the Builder class
Animation.Builder builder = new Animation.Builder();

// Set the origin location (starting point)
builder.setOriginLocation(new ConstantLocation(
    new Location(world, 0.0, 64.0, 0.0)
));

// Set the particle type
builder.setParticleType(Particle.END_ROD);

// Set the particle count (1 particle per frame)
builder.setCount(new ConstantEnvelope(1));

// Build the animation
Animation animation = builder.build();
```

### Step 2: Add Linear Movement

Now let's make the particle move from (0, 64, 0) to (10, 64, 0) over 3 seconds (60 ticks):

```java
import xyz.gameoholic.partigon.particle.envelope.LinearEnvelope;
import xyz.gameoholic.partigon.particle.loop.SingleIteration;
import kotlin.time.Duration;
import kotlin.time.DurationUnit;

Animation.Builder builder = new Animation.Builder();

// Set the origin location
builder.setOriginLocation(new ConstantLocation(
    new Location(world, 0.0, 64.0, 0.0)
));

// Set the particle type
builder.setParticleType(Particle.END_ROD);

// Set the particle count
builder.setCount(new ConstantEnvelope(1));

// Define movement duration (3 seconds = 60 ticks)
int duration = 60;
SingleIteration loop = new SingleIteration(duration);

// Linear movement on X axis: from 0 to 10
builder.setPositionX(new LinearEnvelope(
    new ConstantEnvelope(0.0),      // Start value
    new ConstantEnvelope(10.0),     // End value
    loop
));

// Y and Z remain at 0 (relative to origin)
builder.setPositionY(new ConstantEnvelope(0.0));
builder.setPositionZ(new ConstantEnvelope(0.0));

// Set maximum duration
builder.setMaximumDuration(Duration.Companion.seconds(3));

// Build the animation
Animation animation = builder.build();
```

### Step 3: Start the Animation

To play the animation, you need to provide a CoroutineScope:

```java
import kotlinx.coroutines.CoroutineScope;
import kotlinx.coroutines.CoroutineScopeKt;
import kotlinx.coroutines.Dispatchers;

// Create a coroutine scope
CoroutineScope scope = CoroutineScopeKt.CoroutineScope(Dispatchers.getDefault());

// Start the animation
animation.start(scope);
```

### Step 4: Control the Animation

You can control the animation with these methods:

```java
// Stop the animation
animation.stop();

// Resume from where it stopped
animation.resume(null);

// Resume with a different scope
animation.resume(newScope);
```

## Complete Example

Here's a complete working example that creates a particle moving in a linear path:

```java
import kotlinx.coroutines.CoroutineScope;
import kotlinx.coroutines.CoroutineScopeKt;
import kotlinx.coroutines.Dispatchers;
import org.bukkit.Location;
import org.bukkit.Particle;
import org.bukkit.World;
import xyz.gameoholic.partigon.particle.Animation;
import xyz.gameoholic.partigon.particle.envelope.ConstantEnvelope;
import xyz.gameoholic.partigon.particle.envelope.LinearEnvelope;
import xyz.gameoholic.partigon.particle.location.ConstantLocation;
import xyz.gameoholic.partigon.particle.loop.SingleIteration;
import kotlin.time.Duration;

public class ParticleAnimationExample {
    
    public static Animation createLinearParticleAnimation(World world) {
        Animation.Builder builder = new Animation.Builder();
        
        // Origin point
        builder.setOriginLocation(new ConstantLocation(
            new Location(world, 0.0, 64.0, 0.0)
        ));
        
        // Use glowing particles
        builder.setParticleType(Particle.END_ROD);
        
        // One particle per frame
        builder.setCount(new ConstantEnvelope(1));
        
        // Animation lasts 3 seconds (60 ticks at 20 tps)
        int animationDuration = 60;
        SingleIteration loop = new SingleIteration(animationDuration);
        
        // Move from X=0 to X=10
        builder.setPositionX(new LinearEnvelope(
            new ConstantEnvelope(0.0),
            new ConstantEnvelope(10.0),
            loop
        ));
        
        // Move from Y=0 to Y=5 (creates an arc)
        builder.setPositionY(new LinearEnvelope(
            new ConstantEnvelope(0.0),
            new ConstantEnvelope(5.0),
            loop
        ));
        
        // Stay at Z=0
        builder.setPositionZ(new ConstantEnvelope(0.0));
        
        // Set offsets to 0 (no random spread)
        builder.setOffsetX(new ConstantEnvelope(0.0));
        builder.setOffsetY(new ConstantEnvelope(0.0));
        builder.setOffsetZ(new ConstantEnvelope(0.0));
        
        // Set maximum duration
        builder.setMaximumDuration(Duration.Companion.seconds(3));
        
        return builder.build();
    }
    
    // Usage in a command or event handler:
    public static void playAnimation(World world) {
        Animation animation = createLinearParticleAnimation(world);
        CoroutineScope scope = CoroutineScopeKt.CoroutineScope(Dispatchers.getDefault());
        animation.start(scope);
    }
}
```

## Advanced Tips

### Multiple Particles
To create a trail effect, increase the particle count:

```java
builder.setCount(new ConstantEnvelope(5));  // Spawns 5 particles per frame
```

### Adding Particle Offset
Add randomness to particle position using offsets:

```java
builder.setOffsetX(new ConstantEnvelope(0.5));  // Random spread of 0.5 blocks in X
builder.setOffsetY(new ConstantEnvelope(0.5));
builder.setOffsetZ(new ConstantEnvelope(0.5));
```

### Colored Particles (Dust)
For colored particles, use DustOptions:

```java
import org.bukkit.Color;
import org.bukkit.Particle.DustOptions;

builder.setParticleType(Particle.DUST);
builder.setDustOptions(new DustOptions(Color.RED, 1.0f));  // Red dust, size 1.0
```

### Repeating Animations
To make the animation loop, use RepeatLoop:

```java
import xyz.gameoholic.partigon.particle.loop.RepeatLoop;

RepeatLoop loop = new RepeatLoop(60);  // Repeats every 60 ticks
```

### Animation Interval
Control how often the animation updates:

```java
builder.setAnimationInterval(2);  // Update every 2 ticks instead of every tick
```

## Common Use Cases

### 1. Straight Line (Projectile Trail)
```java
SingleIteration loop = new SingleIteration(100);
builder.setPositionX(new LinearEnvelope(
    new ConstantEnvelope(0.0),
    new ConstantEnvelope(20.0),
    loop
));
builder.setPositionY(new ConstantEnvelope(0.0));
builder.setPositionZ(new ConstantEnvelope(0.0));
```

### 2. Diagonal Movement
```java
SingleIteration loop = new SingleIteration(60);
builder.setPositionX(new LinearEnvelope(
    new ConstantEnvelope(0.0),
    new ConstantEnvelope(10.0),
    loop
));
builder.setPositionY(new LinearEnvelope(
    new ConstantEnvelope(0.0),
    new ConstantEnvelope(10.0),
    loop
));
builder.setPositionZ(new ConstantEnvelope(0.0));
```

### 3. Rising Particle
```java
SingleIteration loop = new SingleIteration(40);
builder.setPositionX(new ConstantEnvelope(0.0));
builder.setPositionY(new LinearEnvelope(
    new ConstantEnvelope(0.0),
    new ConstantEnvelope(5.0),
    loop
));
builder.setPositionZ(new ConstantEnvelope(0.0));
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

**Issues with Kotlin coroutines in Java?**
- Make sure Kotlin standard library and Kotlinx Coroutines are in your classpath
- Use `CoroutineScopeKt.CoroutineScope()` to create scopes from Java
- Use `Dispatchers.getDefault()` for the dispatcher

## Next Steps

Now that you understand basic linear animations, you can explore:
- **MultiAnimation**: Combine multiple animations to play in sync
- **EnvelopeGroups**: Apply transformations to groups of envelopes
- **Rotation**: Add rotation to your particles
- **Trigonometric Envelopes**: Create circular or wave patterns
- **Custom Envelopes**: Use mathematical expressions for complex movements

For more advanced examples, check the source code in the `particle` package.

## Java vs Kotlin

While Partigon is written in Kotlin and has a nice DSL for Kotlin users, it's fully usable from Java. The main differences are:

- **Kotlin DSL**: Uses lambda blocks with `Animation.singleAnimation { ... }`
- **Java Builder**: Uses the Builder pattern with explicit setter methods
- **Extension properties**: Kotlin's `.envelope` becomes `new ConstantEnvelope()` in Java
- **Named parameters**: In Java, you pass parameters in order
- **Coroutines**: You need to use Kotlin's coroutine library from Java

The functionality is identical - just the syntax differs!
