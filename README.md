# exercise 4 (slide 68): uml practice

- [`hospital.md`](hospital.md): exo 1, hospital management system (tasks 1 to 3, class and sequence diagrams, questions)
- [`abcd_records.md`](abcd_records.md): exo 2, abcd records (levels 1 to 3: use case, class and sequence diagrams)

The diagrams are written as text (mermaid) so GitHub draws them. The exercise should also be drawn by hand on paper.

How to read the relations in a class diagram:

| relation | symbol | meaning |
| --- | --- | --- |
| association | simple line or arrow | one object uses or knows another |
| dependency | dashed arrow | a class uses another one temporarily (parameter, local variable) |
| aggregation | hollow diamond | "has a", the part can live without the whole |
| composition | filled diamond | "is made of", the part dies with the whole |
| inheritance | hollow triangle arrow | "is a" (child extends parent) |
| realization | dashed line with hollow triangle | a class implements an interface |

Use cases: **include** = mandatory sub-step (always done), **extend** = optional step (only sometimes).
