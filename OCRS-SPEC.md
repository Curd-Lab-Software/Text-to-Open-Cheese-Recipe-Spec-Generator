# OCRS - Open Cheese Recipe Schema Specification

**Version:** 1.0
**Status:** Draft
**Last Updated:** February 2025

## Overview

OCRS (Open Cheese Recipe Schema) is a structured format for representing artisan cheese recipes in a machine-readable format. It captures all essential information for reproducible cheesemaking, including ingredients, process steps, timing, temperatures, pH targets, and aging requirements.

## Design Principles

1. **Completeness** - Capture all information needed to reproduce a cheese recipe
2. **Flexibility** - Support all cheese styles from fresh to aged
3. **Precision** - Record exact temperatures, times, and pH values where specified
4. **Interoperability** - Use JSON/YAML for easy parsing and exchange
5. **Human Readability** - Maintain clear, descriptive field names

## Schema Structure

```
OCRS Document
├── spec (version identifier)
├── recipe (metadata)
│   ├── name, style, milkType
│   ├── origin, difficulty
│   ├── batchSize, yield
│   ├── totalTime, prepTime
│   └── source
├── ingredients[]
│   ├── name, type, amount, unit
│   └── preparation
├── steps[]
│   ├── stepNumber, title, category
│   ├── instructions
│   ├── temperature, ph, duration
│   └── validation
├── aging
│   ├── required, duration
│   ├── temperature, humidity
│   └── care instructions
├── finalProduct
│   ├── texture, flavor
│   ├── moisture, color
├── equipment
│   ├── required[]
│   └── specialEquipment[]
└── safety
    ├── allergens[]
    ├── shelfLife
    └── storageInstructions
```

## Field Definitions

### Root Object

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `spec` | string | Yes | Schema version, always `"OCRS/1.0"` |
| `recipe` | object | Yes | Recipe metadata |
| `ingredients` | array | Yes | List of ingredients |
| `steps` | array | Yes | Process steps |
| `aging` | object | No | Aging/ripening requirements |
| `finalProduct` | object | No | Expected characteristics |
| `equipment` | object | No | Required equipment |
| `safety` | object | No | Safety and storage info |

### Recipe Metadata

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Recipe name |
| `style` | enum | Yes | Cheese style category |
| `milkType` | enum | Yes | Primary milk source |
| `origin` | string | No | Geographic origin |
| `difficulty` | enum | Yes | Skill level required |
| `batchSize` | object | Yes | Milk volume and unit |
| `yield` | object | No | Expected output |
| `totalTime` | integer | No | Active time in minutes |
| `prepTime` | integer | No | Prep time in minutes |
| `source` | object | No | Recipe attribution |

#### Cheese Styles

| Value | Description | Examples |
|-------|-------------|----------|
| `FRESH` | Unaged, high moisture | Ricotta, Mozzarella, Chevre |
| `SOFT` | Short aging, soft rind | Brie, Camembert |
| `SEMI_SOFT` | Medium moisture | Havarti, Muenster |
| `SEMI_HARD` | Pressed, some aging | Gouda, Cheddar (young) |
| `HARD` | Long aging, low moisture | Parmesan, Aged Cheddar |
| `BLUE` | Blue mold ripened | Roquefort, Gorgonzola |
| `BRINED` | Salt-cured | Feta, Halloumi |

#### Milk Types

| Value | Description |
|-------|-------------|
| `COW` | Bovine milk |
| `GOAT` | Caprine milk |
| `SHEEP` | Ovine milk |
| `BUFFALO` | Water buffalo milk |
| `MIXED` | Blend of milk types |

#### Difficulty Levels

| Level | Criteria |
|-------|----------|
| `BEGINNER` | Fresh cheeses, acid-set, no aging, <3 hours active time |
| `INTERMEDIATE` | Simple cultured, short aging, brining techniques |
| `ADVANCED` | Bloomy/washed rind, long aging, precise temperature/pH control |
| `EXPERT` | Complex multi-culture, extended aging, professional techniques |

### Ingredients

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | Yes | Ingredient name |
| `type` | enum | Yes | Ingredient category |
| `amount` | number | Yes | Quantity |
| `unit` | string | Yes | Unit of measurement |
| `preparation` | string | No | Prep instructions |

#### Ingredient Types

| Value | Description | Examples |
|-------|-------------|----------|
| `PRIMARY` | Base milk | Whole milk, cream |
| `CULTURE` | Starter and mold cultures | Mesophilic, P. candidum |
| `COAGULANT` | Curdling agents | Rennet, citric acid |
| `ADDITIVE` | Process additives | Calcium chloride, lipase |
| `SEASONING` | Flavoring | Salt, herbs, spices |
| `OTHER` | Miscellaneous | Water, wax |

### Steps

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `stepNumber` | integer | Yes | Sequential step number |
| `title` | string | Yes | Brief step title |
| `category` | enum | Yes | Step type category |
| `instructions` | string | Yes | Detailed instructions |
| `temperature` | object | No | Target temperature |
| `ph` | number | No | Target pH value |
| `duration` | object | No | Time requirement |
| `validation` | object | No | Expected outcome |

#### Step Categories

| Value | Description |
|-------|-------------|
| `HEATING` | Warming milk or curds |
| `COAGULATION` | Rennet addition and curd formation |
| `CUTTING` | Cutting the curd mass |
| `DRAINING` | Removing whey |
| `SALTING` | Applying salt (dry or brine) |
| `PRESSING` | Applying weight/pressure |
| `AGING` | Ripening, cave aging |
| `OTHER` | Cheddaring, stretching, washing, etc. |

#### Temperature Object

```json
{
  "target": 32,
  "unit": "C"
}
```

- `target`: Numeric temperature value
- `unit`: `"C"` (Celsius) or `"F"` (Fahrenheit)

#### pH Field

The `ph` field is a simple number representing the target pH for the step:

```json
{
  "stepNumber": 6,
  "title": "Drain Whey",
  "category": "DRAINING",
  "instructions": "Drain whey when pH reaches target",
  "ph": 5.2
}
```

Common pH targets in cheesemaking:
- Fresh milk: 6.5-6.7
- Ripened milk (ready for rennet): 6.4-6.5
- Curd at cutting: 6.3-6.4
- Curd at draining: 5.8-6.2
- Curd at milling/salting: 5.2-5.4
- Final cheese: 4.9-5.3 (varies by style)

#### Duration Object

```json
{
  "type": "FIXED",
  "value": 45,
  "unit": "minutes"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | enum | Yes | Duration type |
| `value` | integer | Conditional | Time value (required for FIXED/RANGE) |
| `unit` | string | Conditional | `"minutes"` or `"hours"` |
| `condition` | string | Conditional | End condition (for UNTIL_CONDITION) |

**Duration Types:**
- `FIXED` - Exact time duration
- `UNTIL_CONDITION` - Wait for specific condition (clean break, pH target)
- `RANGE` - Time window with min/max

### Aging

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `required` | boolean | Yes | Whether aging is needed |
| `duration` | object | Conditional | Aging duration (min/max/unit) |
| `temperature` | object | No | Aging temperature |
| `humidity` | string | No | Humidity range (e.g., "85-90%") |

### Final Product

| Field | Type | Description |
|-------|------|-------------|
| `texture` | string | Expected texture |
| `flavor` | string | Flavor profile |
| `moisture` | enum | `HIGH`, `MEDIUM`, or `LOW` |
| `color` | string | Appearance description |

### Equipment

| Field | Type | Description |
|-------|------|-------------|
| `required` | array | Essential equipment list |
| `specialEquipment` | array | Specialized items with alternatives |

### Safety

| Field | Type | Description |
|-------|------|-------------|
| `allergens` | array | Allergen enum values |
| `shelfLife` | object | Fresh and aged shelf life |
| `storageInstructions` | string | Storage guidance |

**Allergen Values:** `MILK`, `LACTOSE`, `NUTS`, `SOY`, `GLUTEN`, `EGGS`

## Example Document

```yaml
spec: OCRS/1.0
recipe:
  name: Simple Mozzarella
  style: FRESH
  milkType: COW
  difficulty: BEGINNER
  batchSize:
    milkVolume: 3.785
    unit: liters
  yield:
    amount: 450
    unit: grams
  totalTime: 60

ingredients:
  - name: Whole Milk
    type: PRIMARY
    amount: 3.785
    unit: liters
  - name: Citric Acid
    type: ADDITIVE
    amount: 1.5
    unit: tsp
    preparation: Dissolved in 1/4 cup water
  - name: Liquid Rennet
    type: COAGULANT
    amount: 0.25
    unit: tsp
    preparation: Diluted in 1/4 cup water
  - name: Salt
    type: SEASONING
    amount: 1
    unit: tsp

steps:
  - stepNumber: 1
    title: Heat and Acidify
    category: HEATING
    instructions: Heat milk to 13°C, add dissolved citric acid
    temperature:
      target: 13
      unit: C
  - stepNumber: 2
    title: Add Rennet
    category: COAGULATION
    instructions: Heat to 32°C, add diluted rennet
    temperature:
      target: 32
      unit: C
  - stepNumber: 3
    title: Set Curd
    category: COAGULATION
    instructions: Let set until clean break
    duration:
      type: FIXED
      value: 5
      unit: minutes
    validation:
      expectedResult: Clean break when tested
  - stepNumber: 4
    title: Cut Curds
    category: CUTTING
    instructions: Cut into 1-inch cubes
  - stepNumber: 5
    title: Cook Curds
    category: HEATING
    instructions: Heat to 41°C while stirring
    temperature:
      target: 41
      unit: C
  - stepNumber: 6
    title: Drain
    category: DRAINING
    instructions: Drain whey when pH reaches target
    ph: 5.2
    duration:
      type: UNTIL_CONDITION
      condition: pH reaches 5.2
  - stepNumber: 7
    title: Stretch
    category: OTHER
    instructions: Stretch in 77°C water until smooth
    temperature:
      target: 77
      unit: C
  - stepNumber: 8
    title: Salt and Form
    category: SALTING
    instructions: Add salt, form into balls

aging:
  required: false

safety:
  allergens:
    - MILK
    - LACTOSE
  shelfLife:
    fresh: 1 week refrigerated
  storageInstructions: Store in brine or wrapped, refrigerated
```

## Unit Conventions

### Temperature
- Always specify unit: `C` (Celsius) or `F` (Fahrenheit)
- Prefer Celsius for consistency

### Volume
- `liters` or `gallons` for batch size
- `ml`, `tsp`, `tbsp`, `cup` for small quantities

### Weight
- `kg`, `grams` for metric
- `lbs`, `oz` for imperial

### Time
- Store duration values in minutes for calculation
- Display in appropriate units (minutes, hours)

## Validation Rules

1. `spec` must be exactly `"OCRS/1.0"`
2. `recipe.style` must be a valid enum value
3. `ingredients` array must not be empty
4. `steps` array must not be empty
5. `steps[].stepNumber` should be sequential starting from 1
6. If `aging.required` is `true`, `aging.duration` should be present
7. Temperature `unit` must be `"C"` or `"F"`
8. Duration `type` must be `"FIXED"`, `"UNTIL_CONDITION"`, or `"RANGE"`

## File Extensions

- `.ocrs.json` - JSON format
- `.ocrs.yml` or `.ocrs.yaml` - YAML format

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | Feb 2025 | Initial release with pH support |

## License

This specification is released under the MIT License.
