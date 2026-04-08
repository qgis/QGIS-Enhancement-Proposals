# QEP 417: Replace SIP_FACTORY with std::unique_ptr

**Date** 2026/04/08

**Author** Julien Cabieces (@troopa81)

**Contact** julien dot cabieces at oslandia dot com

**Version** QGIS 4.2

# Summary

There are 2 types of methods returning an objet in QGIS API:
- Method caller gets the ownership of the returned object (like all [clone()](https://github.com/qgis/QGIS/blob/d0e537f550fd433e67315ded6706f90cbeb1078d/src/core/geometry/qgscurve.h#L48) methods for instance). The caller is responsible to delete it, otherwise we would get a memory leak.
- Method caller doesn't get the ownership of the returned object (like [QgsProject::mapLayer()](https://github.com/qgis/QGIS/blob/d0e537f550fd433e67315ded6706f90cbeb1078d/src/core/project/qgsproject.h#L1242) for instance). The callee object has still the ownership of the object (`QgsProject` owns `QgsMapLayer` in this example), and the caller **must not** delete it.

This requires an explicit description so that Python, which manages memory differently than C++, knows precisely how to act in each scenario.

QGIS API uses a special macro called SIP_FACTORY for methods which transfer the ownership of the returned object to the caller. This macro is translated as `/Factory/` SIP annotation which tells Python to delete the returned object when it goes out of scope.

On the other hand, QGIS API sometime uses, rightfully, a `std::unique_ptr` as a return type instead of a raw pointer.

Both `std::unique_ptr` and `SIP_FACTORY` describes the same paradigm, making the second redundant. Moreover, QGIS application could benefit from wider use of `std::unique_ptr` to improve memory management.


## Proposed Solution

This QEP proposes to: 
- Modify `sipify.py` script to add the `/Factory/` for every methods whom return type is a `std::unique_ptr`
- Use a `std::unique_ptr` as a return type for every methods which has the `SIP_FACTORY` and remove the use of `SIP_FACTORY` for these methods
- When no methods are using the `SIP_FACTORY` anymore, remove completely the `SIP_FACTORY` definition

As a consequence, all methods already returning a `std::unique_ptr` but missing `SIP_FACTORY` (like [QgsLineSymbol::createSymbol](https://github.com/qgis/QGIS/blob/master/src/core/symbology/qgslinesymbol.h#L36) for instance) would be automatically fixed. At the moment, there are more than 100 of these in QGIS API, **all leading to memory leak when called from a Python context**. 

## How to port the code

There are actually 1700 uses of `SIP_FACTORY`. I'm planning on using a Clang plugin to port the easy ones ([here](https://github.com/qgis/QGIS/blob/d0e537f550fd433e67315ded6706f90cbeb1078d/src/gui/symbology/qgssymbollayerwidget.h#L553) for instance). For the more complex ones, and probably for all those methods calls, rewrite would require to be done manually.

## Deliverables

Pull requests to replace raw pointer returned type with `std::unique_ptr`, and removing `SIP_FACTORY`.

### Example(s)

[Example on QgsProcessingModelAlgorithm](https://github.com/qgis/QGIS/commit/d0e537f550fd433e67315ded6706f90cbeb1078d)

### Affected Files

- All header files containing `SIP_FACTORY`, and the associated *.sip.in*
- `sipify.py`

## Risks

Achieving such migration is complex and we could lack time to complete it. Worst case scenario would be that we fail to migrate all methods, but that would still be a better situation than what we have now.

## Performance Implications

None

## Further Considerations/Improvements

This would be a joined effort with the on going [Security project for QGIS](https://oslandia.com/en/security-project-for-qgis/) and the already proposed [pull requests](https://github.com/qgis/QGIS/pulls?q=is%3Apr+%22Use+unique_ptr+when+class%22+is%3Aclosed) aiming at improving memory safety.


## Backwards Compatibility

This is completely backwards compatible.

## Issue Tracking ID(s)

None
