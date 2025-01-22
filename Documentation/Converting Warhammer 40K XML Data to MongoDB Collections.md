# High-Level Documentation for Converting Warhammer 40K XML Data to MongoDB Collections

## Overview
This document outlines how the XML files from the Warhammer 40K data repository will be parsed, split into distinct MongoDB collections, and structured to maintain relationships between various entities. It incorporates units, rules, constraints, profiles, wargear, and other game elements, enabling a functional and rule-compliant list-building application.

---

## MongoDB Collections and Structure

### 1. `gameSystem`
- **Purpose**: Contains the core game rules and shared definitions applicable to all factions.
- **Schema**:
  ```json
  {
    "_id": "warhammer_40k",
    "name": "Warhammer 40,000",
    "coreRules": ["rule_id_1", "rule_id_2"],
    "categories": ["hq", "troops", "elites"],
    "sharedProfiles": ["profile_id_1", "profile_id_2"]
  }
  ```

### 2. `factions`
- **Purpose**: Represents each faction or sub-faction, containing faction-specific rules and units.
- **Schema**:
  ```json
  {
    "_id": "faction_craftworlds",
    "name": "Craftworlds",
    "factionRules": ["rule_id_3", "rule_id_4"],
    "unitList": ["unit_dire_avengers", "unit_wave_serpent"]
  }
  ```

### 3. `units`
- **Purpose**: Represents individual units, their default composition, wargear, and rules.
- **Schema**:
  ```json
  {
    "_id": "unit_dire_avengers",
    "name": "Dire Avengers",
    "factionId": "faction_craftworlds",
    "baseCost": 65,
    "defaultComposition": {
      "models": [
        {
          "id": "dire_avenger_exarch",
          "name": "Dire Avenger Exarch",
          "quantity": 1,
          "defaultWargear": ["avenger_shuriken_catapult"]
        },
        {
          "id": "dire_avenger",
          "name": "Dire Avenger",
          "quantity": 4,
          "defaultWargear": ["avenger_shuriken_catapult"]
        }
      ]
    },
    "customOptions": {
      "addModels": {
        "min": 5,
        "max": 10,
        "costPerModel": 13
      },
      "wargear": {
        "exarch": {
          "default": ["avenger_shuriken_catapult"],
          "options": [
            {
              "replace": "avenger_shuriken_catapult",
              "choices": ["shuriken_pistol", "diresword", "power_glaive"]
            },
            {
              "replace": "shuriken_pistol",
              "choices": ["shimmershield"]
            }
          ]
        }
      }
    },
    "rules": ["rule_battle_focus"]
  }
  ```

### 4. `models`
- **Purpose**: Represents individual models within a unit and their unique characteristics.
- **Schema**:
  ```json
  {
    "_id": "dire_avenger_exarch",
    "name": "Dire Avenger Exarch",
    "unitId": "unit_dire_avengers",
    "profiles": ["profile_movement", "profile_ballistic_skill"],
    "rules": ["rule_exarch_ability"],
    "wargear": ["shuriken_pistol", "diresword"]
  }
  ```

### 5. `wargear`
- **Purpose**: Contains information about weapons, equipment, and other items available to units.
- **Schema**:
  ```json
  {
    "_id": "shuriken_pistol",
    "name": "Shuriken Pistol",
    "type": "Weapon",
    "profiles": ["profile_shuriken_pistol"],
    "rules": ["rule_shuriken_weapon"]
  }
  ```

### 6. `rules`
- **Purpose**: Stores rules, abilities, and constraints for factions, units, or wargear.
- **Schema**:
  ```json
  {
    "_id": "rule_battle_focus",
    "name": "Battle Focus",
    "description": "Allows models to shoot and then move.",
    "scope": "unit",
    "appliesTo": ["unit_dire_avengers"]
  }
  ```

### 7. `constraints`
- **Purpose**: Manages limits and requirements for units, wargear, and upgrades.
- **Schema**:
  ```json
  {
    "_id": "constraint_max_dire_avengers",
    "type": "maxSelections",
    "scope": "unit",
    "targetId": "unit_dire_avengers",
    "value": 10,
    "conditions": {
      "categories": ["troops"],
      "factionId": "faction_craftworlds"
    }
  }
  ```

### 8. `profiles`
- **Purpose**: Stores stats and characteristics for units, models, and wargear.
- **Schema**:
  ```json
  {
    "_id": "profile_movement",
    "name": "Movement",
    "type": "Model",
    "values": {
      "M": "7\"",
      "T": 3
    }
  }
  ```

### 9. `categories`
- **Purpose**: Defines organizational categories for units (e.g., HQ, Troops, Elites).
- **Schema**:
  ```json
  {
    "_id": "hq",
    "name": "HQ",
    "description": "Headquarters units for the faction.",
    "appliesTo": ["unit_autarch", "unit_farseer"]
  }
  ```

---

## Relationships Between Collections

### Key Relationships:
1. **Factions and Units**:
   - `factions.unitList` references `units._id`.
2. **Units and Models**:
   - `units.defaultComposition.models` references `models._id`.
3. **Units and Rules**:
   - `units.rules` references `rules._id`.
4. **Units and Wargear**:
   - `units.customOptions.wargear` references `wargear._id`.
5. **Models and Profiles**:
   - `models.profiles` references `profiles._id`.
6. **Rules and Constraints**:
   - `rules.appliesTo` references any relevant collection (`units`, `models`, `factions`, etc.).
7. **Constraints and Entities**:
   - `constraints.targetId` references the constrained entity (`units`, `models`, `factions`, etc.).
8. **Profiles and Entities**:
   - `profiles` can be linked to `units`, `models`, or `wargear`.

---

## Summary
This structure modularizes the deeply nested XML data into discrete collections, making it easier to:
- Maintain and query data efficiently.
- Enforce constraints and rules across all levels.
- Provide users with intuitive tools for building and customizing lists.
