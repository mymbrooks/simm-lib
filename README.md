# Simm-Lib

Simm-Lib is an implementation of version 2.0 of the value at risk Standard Initial
Margin Model ([SIMM™ 2.0](https://www2.isda.org/functional-areas/wgmr-implementation/))
developed by ISDA, see [here](https://www2.isda.org/functional-areas/wgmr-implementation/) for methodology specifications. 
It has been built to be compatible with the Common
Risk Interchange Format (CRIF) and it's correlation parameters and risk
weights are interfaced to allow them to be easily exchanged. This makes
Simm-Lib easy to deploy, as most users of Simm-Lib will already be
generating CRIF files; easy to maintain, as the yearly updates to
SIMM™ can be handled with only minor changes to Simm-Lib; and
easy to experiment with, as custom model parameters can be created for
Simm-Lib and implemented with the same minor changes as the yearly updates.

Users of Simm-Lib who wish to deploy it for commercial purposes
will need to obtain a license from ISDA to use ISDA SIMM™ to
calculate initial margin for their or their clients’ non-cleared
derivatives transactions. Please contact isdalegal@isda.org for more
information on licensing the ISDA SIMM™.

### Updates: Change Log
#### 2018-4-4
- Simm-lib has been restructured to be more easily parsed, and also to more easily implement `ImTree` functionality.
As a result, ImTree functionality has been expanded to include Additional-IM sensitivities, and the strict 'Bucket' level
view that the library took previously has been loosened.
- `ImTree` is now an interface, and **no longer** parses directly to the standard IM-Tree CSV format. Instead, a CSV formatted `String`
can be obtained from the `static` method `ImTree.parseToFlatCsv()` which takes an ImTree object as an input.

#### 2017-12-11
- Simm-Lib now includes IM-Tree functionality for non-Additional-IM Sensitivities.
An `ImTree` data structure, which parses directly to the standard IM-Tree CSV format, has been added to carry the IM-Tree structure.
- `ImTree` takes a strict view of the 'Bucket' level of IM-Tree and doesn't distinguish between different
currencies in the FX risk class as all currencies are in the same bucket per ISDA's documentation.
- Simm-Lib has added the Base Correlation Sensitivity Type to the Credit Qualifying risk class.
- Simm-Lib now passes ISDA's Unit Test for SIMM™ 2.0 confirming the accuracy of the calculated exposure.

## Getting Started
Simm-Lib is built with Apache Maven, so one must get Maven
installed on their machine.

Ubuntu users need only run in a terminal:
```
$ sudo apt-get install maven
```

Similarly, Mac users who have [Homebrew](https://brew.sh) installed
can run:
```
$ brew install maven
```

All others can go to the [Maven homepage](https://maven.apache.org)
for specific instructions on how to download and install Maven with
any operating system.

To confirm that Maven has been successfully downloaded, check
the Maven version on your machine by running:
```
$ mvn -version
```
If Maven has been successfully installed, this command should return
something like:
```
Apache Maven 3.5.0 (ff4wa5hff; 2017-04-03T15:39:06-04:00)
Maven home: --MavenHomeDirectory--
Java version: 1.8.0_91, vendor: Oracle Corporation
Java home: --JavaHomeDirectory--
Default locale: en_US, platform encoding: UTF-8
OS name: "mac os x", version: "10.11.6", arch: "x86_64", family: "mac"
```

#### Installing
Simm-Lib's sources have to be moved onto a local machine. This
can be accomplished by either downloading them as a zip file from
GitHub, or by running in a terminal:
```
$ git clone <repo> simm-lib
```
Next, the code's artifacts need to be built. As Simm-Lib is a Maven
project, this process is simplified to running:
```
$ mvn compile
```
After this, Simm-Lib should be fully ready to run in a local
environment.

## Testing
To run all of Simm-Lib's tests, simply run:
```
$ mvn test
```
If one wants to run individual tests, or an individual test module,
the easiest way is to open Simm-Lib in some IDE and run the tests from
that.

#### Testing Breakdown
While there are simple tests to check the functionality of the
individual methods of Simm-Lib, the main focus of the test suite is to
confirm that the outputs of SIMM and Simm-Lib match. To accomplish
this, the tests take in sensitivities and then run Simm-Lib's
calculation, checking the result against SIMM's result:
```java
@Test
public void test() {
    Sensitivity IR1 = new Sensitivity("RatesFX", "Risk_IRCurve", "GBP", "1", "6m", "OIS", new BigDecimal("200000000"));
    Assert.assertEquals(new BigDecimal("13400000000"), Simm.calculateStandard(Arrays.asList(IR1)).setScale(0, RoundingMode.HALF_UP));
}
```
#### Simm Class
This section focuses on the tests for the top level functionality
of the `Simm` class, as the methods of this class directly consume CRIF
formatted data and return the calculated IM of those inputs. There are six
methods in the `Simm` class: `calculateStanard()`, `calculateAdditional()`,
 `calculateTotal()`, `calculateTreeStandard()`, `calculateTreeAdditional()`, and `calculateTreeTotal()`.
The first method returns the IM of the input sensitivities, the second returns the Additional
IM generated by regulatory restrictions (this includes additional value created from product class multipliers),
while the third is the sum of the previous two methods. The "Tree" methods of similar names do the exact same
calculation, except the intermediate exposures at key points are saved in a tree structure to give a more thorough
view of how the exposure was calculated. All methods return the calculated value in US Dollars, so users of
Simm-Lib working in other currencies should be sure to include the "amountCurrency", and "amountUSD" columns in their
CRIF files.

The inputs to these methods are CRIF formatted data-types where all amounts are
stored as `BigDecimal` and all other values as `String`:
```
AddOnNotionalFactor(product, factor)
ProductMultiplier(productClass, multiplier)
AddOnFixedAmount(amount, amountCurrency, amountUSD)
AddOnNotional(product, notional, notionalCurrency, notionalUSD)
Sensitivity(productClass, riskType, qualifier, bucket, label1, label2, amount, amountCurrency, amountUsd)
```
For `Sensitivity`, `AddOnFixedAmount`, and `AddOnNotional` the "currency", and
"amountUSD" can be omitted; however, the currency in this case will be assumed
to be US Dollars.