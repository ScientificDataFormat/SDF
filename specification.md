Scientific Data Format Specification - Draft -
====================================

This document describes how SDF is stored in HDF5.

### Datasets

Datasets may have the following data types: Float, Double and Integer. Every
non-scalar dataset (rank > 0) can have one dimension scale per dimension. The
length of the scale must match the extent of the respective dimension. Elements
must be stored in row-major order.

Variable Type    | HDF5 Data Type | Size
-----------------|:---------------|:----:
Float            | H5T_FLOAT      | 4
Double           | H5T_FLOAT      | 8
Integer          | H5T_INTEGER    | 4

### Dimension Scales

Dimension scales must be one-dimensional and the values must be monotonically
increasing. Dimension scales must not have scales themselves.


### Object Names

The names of Groups and Datasets must only contain upper and lower case letters,
digits and underscores (_) and start with a letter. Object names must conform to
the `object_name` syntax.

```
object_name:
    [a-zA-Z][a-zA-Z0-9_]*
```

### Attributes

Attributes are stored as scalar attributes of type H5T_STRING (variable length).
SDF has the following reserved attribute names:

Name              | Allowed In      | Legal Values    | Description
------------------|-----------------|-----------------|------------
COMMENT           | Group, Dataset  | UTF-8 string    | A short description of the stored quantity
DISPLAY_UNIT      | Dataset         | unit_expression | The display unit of the quantity stored in the dataset
NAME              | Dataset         | UTF-8 string    | The display name of the dataset (shared with the scale_name of the H5DS API)
RELATIVE_QUANTITY | Dataset         | "TRUE"          | Indicates that the stored value is a relative quantity
UNIT              | Dataset         | unit_expression | The unit of the quantity stored in the dataset

- All attributes are optional
- `DISPLAY_UNIT` is only allowed if `UNIT` is present

Attribute names must only contain uppercase letters and digits and start with a
letter. Attribute names must conform to the `attribute_name` syntax.

```
attribute_name:
    [A-Z][A-Z0-9_]*
```

### Units

SDF uses the same unit expressions as Modelica. A unit expression is a string
composed of unit symbols, operands and exponent factors.

#### Operands

Operand        | Symbol | Example
---------------|:------:|:-------:
Multiplication | .      | N.m
Division       | /      | 1/rad
Exponent       | n      | m2

#### Exponent Factors

Name  | Symbol | Factor
------|:------:|----------------
yotta | Y      | 10<sup>24</sup>
zetta | Z      | 10<sup>21</sup>
exa   | E      | 10<sup>18</sup>
peta  | P      | 10<sup>15</sup>
tera  | T      | 10<sup>12</sup>
giga  | G      | 10<sup>9</sup>
mega  | M      | 10<sup>6</sup>
kilo  | k      | 10<sup>3</sup>
hecto | h      | 10<sup>2</sup>
deca  | da     | 10<sup>1</sup>
deci  | d      | 10<sup>-1</sup>
centi | c      | 10<sup>-2</sup>
milli | m      | 10<sup>-3</sup>
micro | u      | 10<sup>-6</sup>
nano  | n      | 10<sup>-9</sup>
pico  | p      | 10<sup>-12</sup>
femto | f      | 10<sup>-15</sup>
atto  | a      | 10<sup>-18</sup>
zepto | z      | 10<sup>-21</sup>
yocto | y      | 10<sup>-24</sup>

Examples are `N.m`, `kg.m/s2`, `kg.m.s-2`, `1/rad`, `mm/s`

#### Expression Syntax

These are the rules to build a unit expression. For a formal definition see the
[Modelica Specification](https://www.modelica.org/documents/ModelicaSpec33Revision1.pdf#page=245) (Chapter 19: Unit Expressions).

```
unit_expression:
    unit_numerator [ "/" unit_denominator ]

unit_numerator:
    "1" | unit_factors |  "(" unit_expression ")"

unit_denominator:
    unit_factor |  "(" unit_expression ")"

unit_factors:
    unit_factor [ unit_mulop  unit_factors ]

unit_mulop:
    "."

unit_factor:
    unit_operand [ unit_exponent ]

unit_exponent:
    [ "+" | "-" ] integer

unit_operand:
    unit_symbol | unit_prefix unit_symbol

unit_prefix:
    Y | Z | E | P | T | G | M | k | h | da | d | c | m | u | n | p | f | a | z | y
```

#### Unit Conversions

This section defines a set of derived units and conversions. Derived units can
be used when plotting presenting data in an UI. The value `B` in the derived
unit is calculated from the value `A` in the base unti as follows:

`B` = `A` * `scale` + `offset`

Likewise `B` can be converted back to `A`:

`A` = (`B` - `offset`) / `scale`

Note that for relative quantities only the scale is applied when converting
between base and derived units.


##### Time

Unit | Derived Unit | Scale
:---:|:------------:|------
s    | ms           | 1000
s    | min          | 0.016666666666666666
s    | h            | 0.0002777777777777778
s    | d            | 1.1574074074074073e-5
s    | m            | 3.80265176e-7


##### Angle

Unit | Derived Unit | Scale
:---:|:------------:|------
rad  | deg          | 57.29577951308232


##### Angular Velocity

Unit | Derived Unit | Scale
:---:|:------------:|------
rad/s| deg/s        | 57.29577951308232
rad/s| rpm          | 9.549296585513721
rad/s| 1/min        | 9.549296585513721
rad/s| r/min        | 9.549296585513721


##### Length & Distance

Unit | Derived Unit | Scale
:---:|:------------:|------
m    | km           | 0.001
m    | cm           | 100
m    | mm           | 1000
m    | ft           | 3.280839895013123
m    | in           | 39.37007874015748


##### Area

Unit | Derived Unit | Scale
:---:|:------------:|------
m2   | cm2          | 1e4


##### Volume

Unit | Derived Unit | Scale
:---:|:------------:|------
m3   | l            | 1e3
m3   | ml           | 1e6


##### Pressure

Unit | Derived Unit | Scale
:---:|:------------:|------
Pa   | kPa          | 1e-3
Pa   | MPa          | 1e-6
Pa   | bar          | 1e-5
Pa   | psi          | 0.00014503774


##### Bulk Modulus

Unit | Derived Unit | Scale
:---:|:------------:|------
N/m2 | bar          | 1e-5


##### Volume Flow Rate

Unit | Derived Unit | Scale
:---:|:------------:|------
m3/s | l/min        | 6e4
m3/s | gal/min      | 15850.330611479338


##### Density

Unit | Derived Unit | Scale
:---:|:------------:|------
kg/m3| kg/dm3       | 1e-3
kg/m3| kg/l         | 1e-3
kg/m3| g/cm3        | 1e-3


##### Mass Flow Rate

Unit | Derived Unit | Scale
:---:|:------------:|------
kg/s | g/s          | 1e3
kg/s | lbm/s        | 2.2046226218


##### Speed

Unit | Derived Unit | Scale
:---:|:------------:|------
m/s  | km/h         | 3.6
m/s  | mm/s         | 1e3
m/s  | knots        | 1.9438445
m/s  | mph          | 2.236941852


##### Force

Unit | Derived Unit | Scale
:---:|:------------:|------
N    | mN           | 1000
N    | kN           | 1e-3
N    | MN           | 1e-6


##### Work & Energy

Unit | Derived Unit | Scale
:---:|:------------:|------
J    | kWh          | 2.7777777777777776e-07
J    | Wh           | 2.7777777777777776e-04
J    | mJ           | 1000
J    | kJ           | 1e-3
J    | MJ           | 1e-6


##### Energy Density

Unit | Derived Unit | Scale
:---:|:------------:|------
J/kg | kJ/kg        | 1e-3
J/kg | MJ/kg        | 1e-6


##### Power

Unit | Derived Unit | Scale
:---:|:------------:|------
W    | mW           | 1000
W    | kW           | 1e-3
W    | MW           | 1e-6


##### Temperature

Unit | Derived Unit | Scale | Offset
:---:|:------------:|-------|-------
K    | degC         | 1     | -273.15
K    | degF         | 1.8   | -459.66999999999996
K    | degR         | 1.8   |


##### Voltage

Unit | Derived Unit | Scale
:---:|:------------:|------
V    | mV           | 1000
V    | kV           | 0.001


##### Currrent

Unit | Derived Unit | Scale
:---:|:------------:|------
A    | mA           | 1000
A    | kA           | 0.001


##### Resistance

Unit | Derived Unit | Scale
:---:|:------------:|------
Ohm  | mOhm         | 1e3
Ohm  | kOhm         | 1e-3


##### Capacitance

Unit | Derived Unit | Scale
:---:|:------------:|------
F    | mF           | 1e3
F    | uF           | 1e6
F    | nF           | 1e9
F    | pF           | 1e12


##### Inductance

Unit | Derived Unit | Scale
:---:|:------------:|------
H    | mH           | 1e3
H    | uH           | 1e6


##### Charge

Unit | Derived Unit | Scale
:---:|:------------:|------
C    | A.h          | 2.7777777777777776e-04


##### Leakage

Unit      | Derived Unit | Scale
:--------:|:------------:|------
m3/(s.Pa) | l/(min.bar)  | 6e9


##### Viscous Friction

Unit        | Derived Unit  | Scale
:----------:|:-------------:|------
N.m/(rad/s) | N.m/(rev/min) | 0.10471975512


##### Kinematic Viscosity

Unit | Derived Unit | Scale
:---:|:------------:|------
m2/s | mm2/s        | 1e6


##### Temperature Coefficient (e.g. of resistance)

Unit | Derived Unit | Scale
:---:|:------------:|------
1/K  | ppm/K        | 1e6


## License

This document is provided "as is" without any warranty. It is licensed under the
[Creative Commons Attribution 4.0 International license](https://creativecommons.org/licenses/by-sa/4.0/).


[Modelica Specification]: https://www.modelica.org/documents/ModelicaSpec33Revision1.pdf
[HDF5 Dimension Scale Specification]: https://www.hdfgroup.org/HDF5/doc/HL/H5DS_Spec.pdf

----------------------------------------------

Copyright &copy; 2017 Dassault Syst&egrave;mes
