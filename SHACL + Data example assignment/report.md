# SHACL Validation Report

## Group 3 - DOISR 2025-2026

This report describes the validation of the sample data created for the Digimon ontology using SHACL.

The validation was performed with the **SHACL Playground**  
(<https://shacl.org/playground/>) using the following files:

- **Shapes graph:** `shapes.ttl`
- **Data graph:** `data.ttl`

After loading both files into the tool, a validation report was generated, which is included in the file `reportShacl.ttl`.

---

## 1. SHACL shapes included

The `shapes.ttl` file contains 9 SHACL shapes, which is within the required range of 5 to 10 shapes. These shapes validate different types of constraints:

1. `dsh:DigimonCoreShape`  
   Checks the basic required information for each Digimon, such as label, stage, type, attribute and memory.

2. `dsh:DigimonStatsShape`  
   Checks that Digimon statistics are present and represented as non-negative integer values.

3. `dsh:DigimonAbilityShape`  
   Checks that each Digimon is linked to at least one move, skill or support skill.

4. `dsh:MoveShape`  
   Checks the structure of moves, including label, move type, power and SP cost.

5. `dsh:AbilityLabelShape`  
   Checks that skills and support skills have a valid non-empty label.

6. `dsh:DigivolutionShape`  
   Checks the structure of digivolutions, including source Digimon, target Digimon and requirements.

7. `dsh:DigivolutionRequirementShape`  
   Checks the numerical requirements needed for digivolution and restricts the allowed properties.

8. `dsh:RequirementUsageShape`  
   Checks that every DigivolutionRequirement is actually linked from at least one Digivolution.

9. `dsh:DigivolvesToShape`  
   Checks that direct `dig:digivolvesTo` relations are only used between Digimon instances.

The shapes cover different SHACL constraint types, including cardinality constraints, datatype constraints, class constraints, value range constraints, logical constraints, closed shapes, node constraints, disjointness and inverse path constraints.

---

## 2. Overall validation result

The validation produced the following overall result:

```ttl
sh:conforms false
```

This result is expected because the `data.ttl` file includes intentionally invalid data in order to test whether the shapes detect errors correctly.

The SHACL Playground generated **27 validation results**. These violations are useful because they show that the shapes correctly identify problems in the data graph.

---

## 3. Evaluation of validation results

### 3.1 Errors in Digimon instances

#### Node: `dig:Brokenmon`

The validation report detected several problems in the `dig:Brokenmon` instance:

| Problem | Constraint violated | Correction |
|---|---|---|
| Empty `rdfs:label` | `sh:minLength` | Add a non-empty label, for example `rdfs:label "Brokenmon"` |
| `dig:hasType` has the literal `"Virus"` instead of an ontology individual | `sh:class dig:DigimonType` | Replace `"Virus"` with `dig:Virus` |
| Missing `dig:hasAttribute` | `sh:minCount` | Add a valid attribute, for example `dig:hasAttribute dig:Fire` |
| Negative memory value | `sh:minInclusive 0` | Replace it with a non-negative integer |
| `dig:hasHP` has the value `"high"` | `sh:datatype xsd:integer` | Replace it with an integer value |
| Negative SP and defense values | `sh:minInclusive 0` | Replace them with non-negative integers |
| Missing attack value | `sh:minCount` | Add exactly one `dig:hasAttack` value |

To correct this resource, all required Digimon properties must be completed and all numerical statistics must be represented as non-negative integers.

Example correction:

```ttl
dig:Brokenmon
    a dig:Digimon ;
    rdfs:label "Brokenmon" ;
    dig:hasStage dig:Rookie ;
    dig:hasType dig:Virus ;
    dig:hasAttribute dig:Fire ;
    dig:hasMemory 4 ;
    dig:hasHP 80 ;
    dig:hasSP 20 ;
    dig:hasAttack 50 ;
    dig:hasDefense 40 ;
    dig:hasIntelligence 35 ;
    dig:hasSpeed 45 ;
    dig:canUseMove dig:PepperBreath .
```

---

### 3.2 Errors in Move instances

#### Node: `dig:BrokenMove`

The validation report detected the following errors:

| Problem | Constraint violated | Correction |
|---|---|---|
| Missing `rdfs:label` | `sh:minCount` | Add exactly one non-empty label |
| Invalid move type: `dig:Vaccine` is not a `dig:MoveType` | `sh:class dig:MoveType` | Use a valid move type, such as `dig:Physical` or `dig:Magic` |
| Negative power value | `sh:minInclusive 0` | Replace it with a non-negative integer |
| `dig:hasSPCost` has the value `"many"` | `sh:datatype xsd:integer` | Replace it with an integer |

To correct this move, the resource must have a valid label, a valid move type and non-negative integer values for power and SP cost.

Example correction:

```ttl
dig:BrokenMove
    a dig:Move ;
    rdfs:label "Broken Move" ;
    dig:hasMoveType dig:Physical ;
    dig:hasPower 20 ;
    dig:hasSPCost 3 .
```

---

### 3.3 Errors in Skill and SupportSkill instances

#### Node: `dig:EmptySkill`

The validation report detected that `dig:EmptySkill` has an empty label.

| Problem | Constraint violated | Correction |
|---|---|---|
| Empty `rdfs:label` | `sh:minLength` | Add a non-empty label |

Example correction:

```ttl
dig:EmptySkill
    a dig:Skill ;
    rdfs:label "Empty Skill" .
```

---

### 3.4 Errors in Digivolution instances

#### Node: `dig:InvalidSelfDigivolution`

The validation report detected two problems:

| Problem | Constraint violated | Correction |
|---|---|---|
| Source Digimon and target Digimon are the same | `sh:disjoint` | Use different individuals for source and target |
| Linked requirement does not conform to the requirement shape | `sh:node` | Link the digivolution to a valid DigivolutionRequirement |

This is important for the digivolution path analysis use case, because a digivolution step should represent a transition from one Digimon to a different Digimon.

Example correction:

```ttl
dig:ValidDigivolutionExample
    a dig:Digivolution ;
    rdfs:label "Agumon to Greymon" ;
    dig:hasSourceDigimon dig:Agumon ;
    dig:hasTargetDigimon dig:Greymon ;
    dig:hasRequirement dig:ValidRequirement .
```

---

### 3.5 Errors in DigivolutionRequirement instances

#### Node: `dig:InvalidRequirement`

The validation report detected several errors:

| Problem | Constraint violated | Correction |
|---|---|---|
| The property `rdfs:comment` is used but not allowed by the closed shape | `sh:closed` | Remove `rdfs:comment` or add it to the shape if it should be allowed |
| Negative minimum level | `sh:minInclusive 0` | Use a non-negative integer |
| `dig:hasMinHP` has the value `"a lot"` | `sh:datatype xsd:integer` | Replace it with an integer |
| Negative minimum SP | `sh:minInclusive 0` | Use a non-negative integer |
| `dig:hasMinAttack` appears more than once | `sh:maxCount 1` | Keep only one value |

The datatype error and the range error for `dig:hasMinHP` appear together because the value is a string instead of an integer. The correction is to replace the value with a valid non-negative integer.

Example correction:

```ttl
dig:InvalidRequirement
    a dig:DigivolutionRequirement ;
    rdfs:label "Corrected requirement" ;
    dig:hasMinLevel 5 ;
    dig:hasMinHP 40 ;
    dig:hasMinSP 1 ;
    dig:hasMinAttack 50 ;
    dig:hasMinDefense 30 ;
    dig:hasMinIntelligence 20 ;
    dig:hasMinSpeed 15 .
```

---

### 3.6 Requirement not linked to any Digivolution

#### Node: `dig:OrphanRequirement`

The validation report detected that `dig:OrphanRequirement` is not used by any `dig:Digivolution` through `dig:hasRequirement`.

| Problem | Constraint violated | Correction |
|---|---|---|
| Requirement is not linked from a Digivolution | inverse path with `sh:minCount` | Link it from at least one Digivolution |

Example correction:

```ttl
dig:SomeDigivolution
    a dig:Digivolution ;
    rdfs:label "Example digivolution" ;
    dig:hasSourceDigimon dig:Agumon ;
    dig:hasTargetDigimon dig:Greymon ;
    dig:hasRequirement dig:OrphanRequirement .
```

---

### 3.7 Digimon without abilities

#### Node: `dig:NoAbilitymon`

The validation report detected that `dig:NoAbilitymon` does not have any move, skill or support skill.

| Problem | Constraint violated | Correction |
|---|---|---|
| No ability relation is present | `sh:or` | Add at least one `dig:canUseMove`, `dig:learnsSkill` or `dig:hasSupportSkill` relation |

Example correction:

```ttl
dig:NoAbilitymon
    dig:canUseMove dig:PepperBreath .
```

---

### 3.8 Invalid direct digivolvesTo relations

#### Nodes: `dig:Agumon` and `dig:UnknownSource`

The validation report detected invalid use of the `dig:digivolvesTo` relation:

| Problem | Constraint violated | Correction |
|---|---|---|
| `dig:Agumon` digivolves to `dig:NotADigimon`, which is not a Digimon | `sh:class dig:Digimon` | Use an object that is an instance of `dig:Digimon` |
| `dig:UnknownSource` is used as subject of `dig:digivolvesTo`, but it is not a Digimon | `sh:targetSubjectsOf` + `sh:class dig:Digimon` | Type the subject as `dig:Digimon` or remove the invalid relation |

Example correction:

```ttl
dig:Agumon dig:digivolvesTo dig:Greymon .
```

If `dig:UnknownSource` is intended to be a Digimon, it should be declared as such and completed with the required Digimon properties.

---

## 4. Corrections to the shapes

No changes were required in the SHACL shapes after the validation. The reported violations correspond to intentionally invalid data included in `data.ttl`.

The shapes behaved as expected because they detected:

- Missing required values;
- Incorrect datatypes;
- Negative numerical values;
- Invalid class assignments;
- Invalid digivolution relations;
- Unused digivolution requirements;
- Disallowed properties in a closed shape.

The validation confirmed that all nine shapes behave correctly: every intentionally invalid instance produced the expected violations, 
while valid instances conformed without errors. No modifications to the shapes were necessary, which indicates that the shape design was sound from the outset.

---