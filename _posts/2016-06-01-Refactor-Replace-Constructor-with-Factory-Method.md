---
layout: post
title: Refactor, Replace Constructor with Factory Method
---

Problem: We are using Type Code instead of Subclases

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

        public void Punch(Person another)
        {
            if (this.HitRate <= 0) return;
            another.ShieldMe(this.HitRate);
        }

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




---
References:

https://sourcemaking.com/refactoring/replace-constructor-with-factory-method
