# In-Depth Documentation for Converting Warhammer 40K XML Data to MongoDB Collections

## Overview
This document provides detailed instructions for converting the XML data from the Warhammer 40K repository into a MongoDB structure. It includes guidelines for splitting data into collections, managing relationships between entities, and enforcing rules, constraints, and customizable configurations within an application.

---

## Goals
1. **Data Organization**: Break down complex XML data into modular, query-efficient MongoDB collections.
2. **Customizability**: Enable users to build and customize lists within the constraints defined by the game rules.
3. **Rule Enforcement**: Ensure points limits, model counts, and other constraints are validated dynamically.

---

## MongoDB Collections and Structure

### 1. `gameSystem`
- **Purpose**: Stores global definitions and rules for the game system.
- **Fields**:
  - `name`: The name of the game system (e.g., "Warhammer 40,000").
  - `coreRules`: A list of rule IDs shared across all factions.
  - `categories`: A list of category IDs (e.g., HQ, Troops, Elites).
  - `sharedProfiles`: A list of profile IDs used globally.
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
- **Purpose**: Defines factions or sub-factions, including their unique rules and units.
- **Fields**:
  - `name`: Faction name (e.g., "Craftworlds").
  - `factionRules`: A list of rule IDs specific to the faction.
  - `unitList`: A list of unit IDs belonging to the faction.
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
- **Purpose**: Represents individual units, their default composition, and available options.
- **Fields**:
  - `name`: Unit name (e.g., "Dire Avengers").
  - `factionId`: Reference to the faction ID this unit belongs to.
  - `baseCost`: The base points cost for the unit.
  - `defaultComposition`: The unit's default configuration, including models and wargear.
  - `customOptions`: Configurable options such as additional models or wargear.
  - `rules`: A list of rule IDs applicable to the unit.
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
- **Purpose**: Stores individual models within a unit and their unique characteristics.
- **Fields**:
  - `name`: Model name (e.g., "Dire Avenger Exarch").
  - `unitId`: Reference to the unit ID this model belongs to.
  - `profiles`: A list of profile IDs containing the model's stats.
  - `rules`: A list of rule IDs specific to the model.
  - `wargear`: A list of wargear IDs equipped by the model.
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
- **Purpose**: Contains information about weapons, equipment, and other items.
- **Fields**:
  - `name`: Wargear name (e.g., "Shuriken Pistol").
  - `type`: Wargear type (e.g., Weapon, Armor).
  - `profiles`: A list of profile IDs related to the wargear.
  - `rules`: A list of rule IDs applied to the wargear.
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
- **Purpose**: Defines rules and abilities applicable to factions, units, or wargear.
- **Fields**:
  - `name`: Rule name (e.g., "Battle Focus").
  - `description`: Text description of the rule.
  - `scope`: Scope of the rule (e.g., Global, Faction, Unit, Wargear).
  - `appliesTo`: A list of references to entities the rule applies to.
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
- **Purpose**: Manages limits and requirements for entities.
- **Fields**:
  - `type`: Type of constraint (e.g., `maxSelections`, `minSelections`, `pointsLimit`).
  - `scope`: The level at which the constraint applies (e.g., `global`, `unit`, `model`).
  - `targetId`: ID of the entity the constraint applies to.
  - `value`: Numeric value defining the limit.
  - `conditions`: Optional conditions for applying the constraint.
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
- **Purpose**: Stores statistical profiles for units, models, or wargear.
- **Fields**:
  - `name`: Profile name (e.g., "Movement").
  - `type`: Type of profile (e.g., Unit, Model, Wargear).
  - `values`: Key-value pairs for stats (e.g., `{ "M": "7\"", "T": 3 }`).
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
- **Purpose**: Groups units into classifications (e.g., HQ, Troops, Elites).
- **Fields**:
  - `name`: Category name.
  - `description`: Description of the category.
  - `appliesTo`: List of entity IDs the category applies to.
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

### Key Relationships
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
   - `rules.appliesTo` references entities (`units`, `models`, etc.).
7. **Constraints and Entities**:
   - `constraints.targetId` references constrained entities.
8. **Profiles and Entities**:
   - `profiles` link to `units`, `models`, or `wargear`.

---

## Implementation Notes

### Parsing XML
- Use Python libraries like `xml.etree.ElementTree` or `lxml` to parse XML files.
- Normalize relationships (e.g., resolve `EntryLinks` and `SelectionGroups`).
- Flatten deeply nested structures into modular JSON objects.

### Validation Logic
- Enforce constraints during list-building operations.
- Validate points totals, model counts, and dependencies dynamically.

### Real-Time Updates
- Provide users with instant feedback on invalid selections.
- Recalculate points and enforce constraints as units or wargear are added/removed.

---

## Summary
This detailed documentation covers the structure, relationships, and workflows needed to transform Warhammer 40K XML data into MongoDB collections. It ensures modularity, query efficiency, and compliance with game rules. For further implementation or code samples, feel free to ask!

