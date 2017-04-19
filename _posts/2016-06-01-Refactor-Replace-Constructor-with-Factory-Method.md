---
layout: post
title: Refactor, Replace Constructor with Factory Method
description: Notes about abutomatic guided refactoring 
date: 2016-06-01
categories: technology update
img: refactoring.jpg
author: Carlos Raffellini
---

### Problem: We are using Type Code instead of Subclases

```C#
    [TestClass]
    public class Scenario
    {
        [TestMethod]
        public void Soldier_punch_village()
        {
            var village = new Person(0, 0);
            var soldier = new Person(10, 10);
            soldier.Punch(village);
            Assert.AreEqual(0, village.Life);
        }

        [TestMethod]
        public void Commando_shot_sniper()
        {
            var commando = new Person(50, 50);
            var sniper = new Person(1000, 10);
            commando.Punch(sniper);
            Assert.AreEqual(60, sniper.Life);
        }
    }
    
    public class Person
    {
        private int HitRate { get; set; }
        private int ShieldRate { get; set; }
        public int Life { get; set; } = 100;

        public Person(int hitRate, int shieldRate)
        {
            HitRate = hitRate;
            ShieldRate = shieldRate;
        }

        //If's used here show you that this is Type Code and could be replaced with subclasses
        public void Punch(Person another)
        {
            if (this.HitRate <= 0) throw new Exception("Village can not punch");
            another.ShieldMe(this.HitRate);
        }

        //If's used here show you that this is Type Code and could be replaced with subclasses
        private void ShieldMe(int hitRate)
        {
            if (this.ShieldRate == 0) this.Life = 0;
            else
            {
                var effect = Math.Max(0, hitRate - this.ShieldRate);
                this.Life -= effect;
            }
        }
    }
```

### Used Refactors:
- **Replace Constructor with Factory Method**
- and then **Replace Type Code with Subclasses** inside the Factory Method

```C#
    [TestClass]
    public class Scenario
    {
        [TestMethod]
        public void Soldier_punch_village()
        {
            var village = Person.CreatePerson(0, 0);
            var soldier = Person.CreatePerson(10, 10);
            soldier.Punch(village);
            Assert.AreEqual(0, village.Life);
        }

        [TestMethod]
        public void Commando_shot_sniper()
        {
            var commando = Person.CreatePerson(50, 50);
            var sniper = Person.CreatePerson(1000, 10);
            commando.Punch(sniper);
            Assert.AreEqual(60, sniper.Life);
        }
    }
    
    public abstract class Person
    {
        //This is the Factory Method but instad of just pass the arguments to
        //the old constructor Person(int, int), it chose between the new subclasses
        //and instantiate the correct one.
        public static Person CreatePerson(int hitRate, int shieldRate)
        {
            if (shieldRate > 0) return new Soldier(hitRate, shieldRate);
            else return new Village();
        }

        protected int HitRate { get; set; }
        protected int ShieldRate { get; set; }
        public int Life { get; set; } = 100;
        protected abstract void ShieldMe(int hitRate);
        public abstract void Punch(Person another);

        public class Village : Person
        {
            public override void Punch(Person another)
            {
                throw new Exception("Village can not punch");
            }

            protected override void ShieldMe(int hitRate)
            {
                this.Life = 0;
            }
        }

        public class Soldier : Person
        {
            public Soldier(int hitRate, int shieldRate)
            {
                HitRate = hitRate;
                ShieldRate = shieldRate;
            }

            public override void Punch(Person another)
            {
                another.ShieldMe(this.HitRate);
            }

            protected override void ShieldMe(int hitRate)
            {
                var effect = Math.Max(0, hitRate - this.ShieldRate);
                this.Life -= effect;
            }
        }
    }
```

####### You can run it with C# 6 and _Microsoft.VisualStudio.TestTools.UnitTesting_ library.

---
References:

https://sourcemaking.com/refactoring/replace-constructor-with-factory-method
