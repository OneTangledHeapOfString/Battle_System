// Character model (in battle only)
// Do not read yet
function unit_model(unit_data) {
    
    this.Basedamage = unit_data.getBaseDamage();
    this.Swingdamage = unit_data.getSwingDamage();
    
    this.DEF = unit_data.getDEF();
    this.TotalDEF = this.DEF; // Uninitialised TotalDEF
    this.FireRES = unit_data.getRES("fire");
    this.AquaRES = unit_data.getRES("aqua");
    this.LightningRES = unit_data.getRES("lightning");
    
    this.Accuracy = unit_data.getAccuracy();
    this.Evasion = unit_data.getEvasion();
    this.BlockRate = unit_data.getBlockRate();
    this.PierceRate = unit_data.getPierceRate();
    
    this.MaxHP = unit_data.getMaxHP();
    this.HP = unit_data.getHP();
    
    this.BuffList = unit_data.getBuffList();
}

unit_model.prototype.BuffLevel = function(BuffName) {
    // Do check
}

unit_model.prototype.removeFromField = function() {
    // REMOVE YOURSELF
}

// For example, if we want to have weapons.
// First, we must have a database of weapons (an array of (standard) weapons
var weapon_database = [];

// A standard weapon is made from a weapon model
function weapon_model(Name, ID, Basedamage, Swingdamage , Class, Description) {
    this.name = Name;
    this.ID = ID;
    this.basedamage = Basedamage;
    this.swingdamage = Swingdamage;
    this._class = Class; // class is a special keyword
    this._description = Description; // description is a special keyword
}

// We put (standard) weapons into the database
weapon_database[0] = new weapon_model("The Laxifier", 0, Infinity, 0, "Lax-er", "Ultimate weapon of Laxification");
weapon_database[1]= new weapon_model("Sniper Cannon", 1, 2000, 150 , "Rifle", "Long range OPness");

// Non-OOP function
function fake_pow(base, EXP) {
    var a = base; // match the top
    var b = EXP; // It's not necessary to declare new variables for these though
    
    var c = base + EXP; // You can use it directly as it is
    var d = a + b; // Or use the re-declared ones
}

//Damage Formula
// Sequence
// Can you reach the target?

// Can you hit - if you miss damage = 0
// Can he block - if he does his DEF increases
// Skip if Attack has No Dodge/No Block attribute
// Damage is calculated
// Target's HP is subtracted according to Damage

// To makes things easier - shorten code
function make_dmg(User, mul) {
    return (User.Basedamage + User.Swingdamage * math.random()) * mul;
}

function make_def(User, Target, dmg_table, Skill) {
    
    var raw_physical = dmg_table.physical;
    var raw_fire = dmg_table.fire;
    var raw_aqua = dmg_table.aqua;
    var raw_lightning = dmg_table.lightning;
    
    // Formulaic Reference
    // If Physical, Modified Damage = User.Damage - Target.DEF
    // If Elemental, Modified Damage = User.Damage - Target.DEF * (TargetRespectiveRES / User.RespectiveParticleCountOfAttack)
    // Particle Count is how many Particle Units were used in the magical attack (For example, if Spark Bolt uses 30 PU of Lightning Particles
    // Then User.RespectiveParticleCount = 30;)
    // Target.RespectiveRES can be negative resulting in additional damage, where DEF works against the Target
    var modified_physical = raw_physical - Target.TotalDEF;
    var modified_fire = raw_fire - Target.TotalDEF * (Target.FireRES / Skill.FireParticle);
    var modified_aqua = raw_aqua - Target.TotalDEF * (Target.AquaRES / Skill.AquaParticle);
    var modified_lightning = raw_lightning - Target.TotalDEF * (Target.LightningRES / Skill.LightningParticle);
    
    var modified_damage_table = {"physical" : modified_physical, "fire" : modified_fire,
                                 "aqua" : modified_aqua, "lightning" : modified_lightning};
                                 
    return modified_damage_table;
}
    
function ApplyDamage(User, Target, Skill) {
   var PhysicalDamage = make_dmg(User, Skill.PhysicalMultiplier); //Dictates raw damage first
   var FireDamage = make_dmg(User, Skill.FireMultiplier);
   var AquaDamage = make_dmg(User, Skill.AquaMultiplier);
   var LightningDamage = make_dmg(User, Skill.LightningMultiplier);
   
   var Damage_Table = {"physical" : PhysicalDamage, "fire" : FireDamage, "aqua" : AquaDamage, "lightning" : LightningDamage};
   
   var modified_damage = make_def(User, Target, Damage_Table, Skill);
   
   var total_dmg = Math.ceil(modified_damage.physical + modified_damage.fire + modified_damage.aqua + modified_damage.lightning);
   Target.HP = Target.HP - total_dmg;
   //Print total_dmg number
}

function EvadeCheck(User, Target) { // We pass in the User and Target object
   var Random = Math.random();
   var HitChance = User.Accuracy / Target.Evasion; // We grab the User's Accuracy and the Target's Evasion
   if (HitChance > Random) { //Does the attack connect?
        return true;
   } else {
       return false;
   }
}

// Relevant stats: Target.DEF, Target.TotalDEF, Target.BlockDEF, Target.BlockRate, User.PierceRate
function BlockCheck(User, Target, Skill) {
    var Random = Math.random() * 100;
    var BlockChance = Target.BlockRate - User.PierceRate; //Subtraction. BlockChance can be negative
    if (Random > BlockChance && Skill.Blockable) {
        Target.TotalDEF = Target.DEF + Target.BlockDEF; // DEF is raised by adding Target's DEF with the BlockDEF of the equipped shield
    } else {
        Target.TotalDEF = Target.DEF; // Block Failure, no DEF bonus
    }
}

function Attack(User, Target, Skill) {
    
    // Check if you could hit
    var managed_to_hit = EvadeCheck(User, Target);
    
    if (managed_to_hit) {
        
        // Modify Target's Total Defence
        BlockCheck(User, Target, Skill);
        
        // Apply Damage
        ApplyDamage(User, Target, Skill);
        
        var preserve_threshold = -Target.BuffLevel("Preserve") * Target.MaxHP * 0.1; //Every level of Preserve allows HP to go 10% below 0
        
        if (Target.HP <= preserve_threshold) { // Also if a target on Preserve is felled it will be immediately removed
        
            if (preserve_threshold !== 0) { // Had preserve
                Target.removeFromField(); // Calls method of target to remove himself; REMOVE YOURSELF <- LOL
            } else {
                Target.Unconscious = true; // Wait to be tickled. HOPE IS FUTILE
                Target.HP = 0;
            }
        } else {
            // Dance around like a drunk Lax
        }

    } else {
        // Do nothing
        // Print out "Miss"
        // Pump Accuracy for Othos' sake
        // Or just use Lightning you lazy shit
    }
}

// Check if some buff is in list of buffs
// List of buffs should be an array
function has(list_of_buffs, buffs_to_check_for) {
    
    var check = false;
    
    for (var i = 0; i < list_of_buffs.length; i = i + 1) {
        if (list_of_buffs[i] === buffs_to_check_for) {
            check = true;
        } else { // Do nothing 
        }
    }
    
    return check;
}

// Healing can be done by Skills or Items!
// Toxify 'clouds' the upper part of the HP bar preventing recovery up to a certain %; 50% clouding at lv5. Nasty.
// Can be programmed in a similar way as Preserve - if incoming healing would touch the cloud, cap it at the cloud
// MaxHP of target is NOT AFFECTED
function Heal(User, Target, Skill) {
    if (has(User.BuffList, "Toxify")) {
        // Cut heal
    } else {
        // Regular Heal
    }
}
