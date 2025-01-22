## Database Schema for XML Mappings from `.cat` and `.gst` Files

### Overview
The database schema is designed to:
1. Accurately represent the hierarchical and relational structures found in `.cat` and `.gst` files.
2. Facilitate efficient querying for the web front-end, ensuring simple and direct queries.
3. Centralize shared data and maintain references to reduce duplication.

The schema uses **MongoDB** as the database system due to its flexibility in handling hierarchical data and its compatibility with JSON-like structures.

---

### Collections

#### 1. `gameSystems`
- **Purpose**: Store metadata and shared elements from `.gst` files.
- **Structure**:
  ```json
  {
    "_id": "<game_system_id>",
    "name": "<game_system_name>",
    "revision": "<revision_number>",
    "author": "<author_name>",
    "rules": [
      {
        "_id": "<rule_id>",
        "name": "<rule_name>",
        "description": "<rule_description>"
      }
    ],
    "profiles": [
      {
        "_id": "<profile_id>",
        "name": "<profile_name>",
        "type": "<profile_type>",
        "characteristics": [
          {"name": "<char_name>", "value": "<char_value>"}
        ]
      }
    ]
  }
  ```

#### 2. `factions`
- **Purpose**: Store data specific to factions, including units, wargear, and rules.
- **Structure**:
  ```json
  {
    "_id": "<faction_id>",
    "name": "<faction_name>",
    "gameSystemId": "<game_system_id>",
    "units": [
      {
        "_id": "<unit_id>",
        "name": "<unit_name>",
        "profiles": ["<profile_id>"],
        "categories": ["<category_id>"],
        "wargear": ["<wargear_id>"]
      }
    ],
    "wargear": [
      {
        "_id": "<wargear_id>",
        "name": "<wargear_name>",
        "profiles": ["<profile_id>"],
        "rules": ["<rule_id>"]
      }
    ],
    "rules": ["<rule_id>"],
    "categories": [
      {
        "_id": "<category_id>",
        "name": "<category_name>"
      }
    ],
    "constraints": [
      {
        "_id": "<constraint_id>",
        "type": "<constraint_type>",
        "scope": "<constraint_scope>",
        "value": "<constraint_value>"
      }
    ]
  }
  ```

#### 3. `units`
- **Purpose**: Store detailed unit data for direct queries.
- **Structure**:
  ```json
  {
    "_id": "<unit_id>",
    "name": "<unit_name>",
    "factionId": "<faction_id>",
    "profiles": ["<profile_id>"],
    "wargear": ["<wargear_id>"],
    "rules": ["<rule_id>"],
    "categories": ["<category_id>"]
  }
  ```

#### 4. `wargear`
- **Purpose**: Store standalone wargear data.
- **Structure**:
  ```json
  {
    "_id": "<wargear_id>",
    "name": "<wargear_name>",
    "profiles": ["<profile_id>"],
    "rules": ["<rule_id>"],
    "constraints": ["<constraint_id>"]
  }
  ```

#### 5. `rules`
- **Purpose**: Centralize rules for easy access.
- **Structure**:
  ```json
  {
    "_id": "<rule_id>",
    "name": "<rule_name>",
    "description": "<rule_description>"
  }
  ```

#### 6. `profiles`
- **Purpose**: Store detailed profile data.
- **Structure**:
  ```json
  {
    "_id": "<profile_id>",
    "name": "<profile_name>",
    "type": "<profile_type>",
    "characteristics": [
      {"name": "<char_name>", "value": "<char_value>"}
    ]
  }
  ```

#### 7. `categories`
- **Purpose**: Centralize categories for referencing.
- **Structure**:
  ```json
  {
    "_id": "<category_id>",
    "name": "<category_name>"
  }
  ```

#### 8. `constraints`
- **Purpose**: Store constraints applicable to units, wargear, or factions.
- **Structure**:
  ```json
  {
    "_id": "<constraint_id>",
    "type": "<constraint_type>",
    "scope": "<constraint_scope>",
    "value": "<constraint_value>",
    "conditions": [
      {"name": "<condition_name>", "value": "<condition_value>"}
    ]
  }
  ```

#### 9. `modifiers`
- **Purpose**: Store modifiers for specific entries.
- **Structure**:
  ```json
  {
    "_id": "<modifier_id>",
    "field": "<field_name>",
    "operation": "<operation_type>",
    "value": "<modifier_value>"
  }
  ```

---

### Query Examples for Web Front-End

#### 1. **Fetch All Units for a Faction**
```json
{
  "factionId": "<faction_id>"
}
```

#### 2. **Fetch Profiles for a Unit**
```json
{
  "_id": "<unit_id>"
}
```

#### 3. **Fetch Rules for a Faction**
```json
{
  "factionId": "<faction_id>"
}
```

#### 4. **Fetch Wargear for a Unit**
```json
{
  "unitId": "<unit_id>"
}
```

---

### Benefits of This Schema
1. **Centralization**: Shared elements (e.g., rules, profiles) are stored once and referenced, avoiding duplication.
2. **Simplicity**: Queries are straightforward, targeting specific collections.
3. **Scalability**: The schema accommodates future additions without significant refactoring.
4. **Normalization**: Data is organized logically to reflect the XML structure while maintaining flexibility.

Let me know if you need further adjustments or optimizations!

