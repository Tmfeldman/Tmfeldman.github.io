---
layout: post
title:  "Using Mixins with Minecraft Forge 1.21"
date:   2024-10-07 1:36:00 -0500
categories: minecraft forge mixins modding
---

## Introduction

Mixins are a powerful tool that allow mod authors to modify existing Minecraft code without overwriting whole classes or methods. Using mixins reduces the risk of mod incompatibility by allowing mod authors to precisely target specific code blocks. Additionally, using mixins can be quite a fun a rewarding experience. 

Using mixins with the Fabric Mod loader is quite simple and well documented. However, mixins can be quite confusing to set up for the Forge Mod loader if you haven't done it before. Configuring mixins with Forge is simple if you know what you're doing. Unfortunately, there is a lack of documentation for setting up mixins with Forge, specifically for Forge 1.21. In this post we'll go over how to set up mixins for Forge, and write our first mixin.


## Prerequisites 

In order to use mixins, you must first create your own minecraft mod which will contain the mixins. To create a mod you can follow along with [Kaupenjoe's forge 1.21 modding tutorial](https://www.youtube.com/watch?v=eFofdJ1BYYs). Setting all of this up will likely take around 30 minutes from scratch. If you don't want to take time to do that, Kaupenjoe has made the repo available publicly [here](https://github.com/Tutorials-By-Kaupenjoe/Forge-Tutorial-1.21.X/tree/1-setup).  If you are new to modding or don't have Intellij properly set up, I recommend you follow along with the video to create you mod. However, In this tutorial I will be pulling Kaupenjoe's repo. If you do follow the video, you can skip to [Configuring Mixins](#configuring-mixins)

## Setting up your repo

Before we can begin to use mixins, we need to create our own repo with all of the base Forge 1.21 mod files. To do this we will be creating our own copy of [Kaupenjoe's Forge 1.21 tutorialmod](https://github.com/Tutorials-By-Kaupenjoe/Forge-Tutorial-1.21.X/tree/1-setup)

First, login to [github](https://github.com) and create a new repo. I named my repo [Forge1.21Tutorial-mixin](https://github.com/Tmfeldman/Forge1.21Tutorial-mixin), but you can name this repo whatever you want.

Pull  Kaupenjoe's repo and push to your new repo:
{% highlight bash %}
git clone -b 1-setup --single-branch https://github.com/Tutorials-By-Kaupenjoe/Forge-Tutorial-1.21.X.git
cd Forge-Tutorial-1.21.X
git push https://github.com/<YOUR_USERNAME>/Forge1.21Tutorial-mixin  +1-setup:main
cd ..
git clone https://github.com/<YOUR_USERNAME>/Forge1.21Tutorial-mixin.git
cd Forge1.21Tutorial-mixin
{% endhighlight %}

Your repo is now set up!

## Setting up Intellij

Open your repo in Intellij. The first time you open it, you may need to wait a few minutes for Gradel to build. Once Gradel has finished building, open up the Gradel panel on the right sidebar. Navigate to `Tasks/forgegradle runs/runClient` and double click. This should open up a development minecraft instance. If this works then Intellij is configured correctly. If you are having issues, you may need to follow [Kaupenjoe's forge 1.21 modding tutorial](https://www.youtube.com/watch?v=eFofdJ1BYYs) to set up Intellij.

## Configuring Mixins

To configure our mod to use mixins, all we need to do is add a few lines to build.gradel, and create a config file.

# Updating build.gradel

Open build.gradel in Intellij.

Add this line to the plugins block at the top of the file `id 'org.spongepowered.mixin' version '0.7.+'`

Add this line to the dependencies block near the bottom of the file `annotationProcessor 'org.spongepowered:mixin:0.8.5:processor'`

Add this line `'MixinConfigs'            : "mixins.${mod_id}.json"` to the attributes array 
{% highlight ruby %}
tasks.named('jar', Jar).configure {
    manifest {
        attributes([
				....
		  ])
    }
}
{% endhighlight %}

Finally, add this block underneath the dependencies block. And click the load Gradel changes icon on the top right. 
{% highlight ruby %}
mixin {
    // MixinGradle Settings
    add sourceSets.main, 'mixins.tutorialmod.refmap.json'
    config 'mixins.tutorialmod.json'
    debug.verbose = true
    debug.export = true
}
{% endhighlight %}

# Setting up config file

Create a new file at the following path `src/main/resources/mixins.tutorialmod.json`

Set the contents of this file to 
{% highlight ruby %}
{
  "required": true,
  "compatibilityLevel": "JAVA_21",
  "package": "net.kaupenjoe.tutorialmod.mixin",
  "refmap": "tutorialmod.mixin-refmap.json",
  "injectors": {
    "defaultRequire": 1
  },
  "mixins": [
  ]
}
{% endhighlight %}

On the right sidebar, run `Tasks/forgegradle runs/genIntellijRuns`

We are now ready to start using mixins!

## Our First Mixins

To demonstrate the power of mixins, we will inject some code into Minecraft to allow the player to jump higher.

First, click the download sources button at the top of the Gradel sidebar. This will allow us to inspect Minecraft's source code so that we can inject into it. 

Create a directory called mixin at this location in your repo `java/net/kaupenjoe/tutorialmod/mixin`

Create a file `TutorialModMixin.java` inside of the `mixin` folder

Open `TutorialModMixin.java` and add these lines 
{% highlight ruby %}
package net.kaupenjoe.tutorialmod.mixin;

import net.minecraft.world.entity.LivingEntity;
import net.minecraft.world.entity.player.Player;

import org.spongepowered.asm.mixin.Mixin;
import org.spongepowered.asm.mixin.injection.At;
import org.spongepowered.asm.mixin.injection.Inject;
import org.spongepowered.asm.mixin.injection.callback.CallbackInfoReturnable;
{% endhighlight %}

# Identifying Our Target

Now CTRL-LeftClick on `Player` to bring up the de-compiled code for the Player class. Inside of the player class we see the method `jumpFromGround` which we need to modify in order to boost our jump height. 

{% highlight ruby %}
    @Override
    public void jumpFromGround() {
        super.jumpFromGround();
        this.awardStat(Stats.JUMP);
        if (this.isSprinting()) {
            this.causeFoodExhaustion(0.2F);
        } else {
            this.causeFoodExhaustion(0.05F);
        }
    }
{% endhighlight %}

We can see that `jumpFromGround` calls a method `super.jumpFromGround()` which handles the jump. CTRL-LeftClick on `super.jumpFromGround()` and we are brought to the `LivingEntity` class with the following methods.

{% highlight ruby %}
    protected float getJumpPower() {
        return this.getJumpPower(1.0F);
    }

    protected float getJumpPower(float pMultiplier) {
        return (float)this.getAttributeValue(Attributes.JUMP_STRENGTH) * pMultiplier * this.getBlockJumpFactor() + this.getJumpBoostPower();
    }

    @VisibleForTesting
    public void jumpFromGround() {
        float f = this.getJumpPower();
        if (!(f <= 1.0E-5F)) {
            Vec3 vec3 = this.getDeltaMovement();
            this.setDeltaMovement(vec3.x, (double)f, vec3.z);
            if (this.isSprinting()) {
                float f1 = this.getYRot() * (float) (Math.PI / 180.0);
                this.addDeltaMovement(new Vec3((double)(-Mth.sin(f1)) * 0.2, 0.0, (double)Mth.cos(f1) * 0.2));
            }

            this.hasImpulse = true;
            net.minecraftforge.common.ForgeHooks.onLivingJump(this);
        }
    }
{% endhighlight %}

From here we can see that we need to modify `getJumpPower` if we want the player to jump higher. Because the `getJumpPower` method lives in `LivingEntity`, our mixin will need to inject into `LivingEntity`.

# Injecting Into Our Target 

Add the following lines to `TutorialModMixin.java`. 

{% highlight ruby %}
@Mixin(LivingEntity.class)
public abstract class TutorialModMixin {
    @Inject(method = "getJumpPower(F)F", at = @At("RETURN"), cancellable = true)
    protected void MultiplyJumpPower(CallbackInfoReturnable<Float> power) {
        
    }
}
{% endhighlight %}

`@Mixin(LivingEntity.class)` tells mixin that we want to target the `LivingEntity` class. 

`@Inject(method = "getJumpPower(F)F"` tells mixin to look for a function `getJumpPower` that takes `F(Float)` as input and returns `F(Float)`.

`at = @At("RETURN")` tells mixin to inject our code when the function `getJumpPower` is returning. 

`protected void MultiplyJumpPower(CallbackInfoReturnable<Float> power) {}` is the function which will run when `getJumpPower` is called and then returns. Any code we define inside of this function will be run every time `getJumpPower` returns.

To boost the jump height of a player, we simply increase the value of everything returned from `getJumpPower`. To do this, we can add the following line of code to our function `MultiplyJumpPower`. I increased the jump power by 3, but you can increase it as much as you want. 

{% highlight ruby %}
@Mixin(LivingEntity.class)
public abstract class TutorialModMixin {
    @Inject(method = "getJumpPower(F)F", at = @At("RETURN"), cancellable = true)
    protected void MultiplyJumpPower(CallbackInfoReturnable<Float> power) {
        power.setReturnValue(power.getReturnValueF()*3);
    }
}
{% endhighlight %}

The last step is to limit our jump boost to only players. Because we are injecting into the `LivingEntity`, all living entities will receive our jump boost. To limit this boost only to players, we can updated our mixin one last time to the following.

{% highlight ruby %}
@Mixin(LivingEntity.class)
public abstract class TutorialModMixin {
    @Inject(method = "getJumpPower(F)F", at = @At("RETURN"), cancellable = true)
    protected void MultiplyJumpPower(CallbackInfoReturnable<Float> power) {
        if ((LivingEntity)(Object)this instanceof Player){
            power.setReturnValue(power.getReturnValueF()*3);
        }
    }
}
{% endhighlight %}

Finally add `"TutorialModMixin"` to the `mixin` section of `mixins.tutorialmod.json`

And you're done! Launch the dev minecraft instance with `Tasks/forgegradle runs/runClient` and enjoy your new jumping powers.

## References

If you had trouble following along, here is the code I used when following along with this tutorial [GitHub](https://github.com/Tmfeldman/Forge1.21Tutorial-mixin). I found this [Guide](https://fabricmc.net/wiki/tutorial:mixin_examples) helpful while writing mixins. It is written for Fabric, but it works for Forge as well at least most of the time.



