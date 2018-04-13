# UFO Checks
This project is a prototype collection of data for checking UFO fonts according to multiple best practice guidelines. It is intended as a first step toward a flexible specification that can be built into font checking frameworks such as [fontbakery](https://github.com/googlefonts/fontbakery) and [pysilfont](https://github.com/silnrsi/pysilfont). It goes beyond the basic lint-style checking of [ufolint](https://github.com/source-foundry/ufolint).

The format of data used will likely change and evolve, so it may be premature to build this data and format into tools at this point. The specification below is also neither formal nor complete. __You've been warned!__

*Please participate in forming this specification. The best way to do that is to create github issues, and respond to issues that others have begun. Thank you!*

# Goals
To adequately meet the needs of the broad community, this project will need to:

- Be flexible enough to be used by multiple tools and frameworks. It should provide information in an easily parsable and useful format.
- Support diverse specifications for what is considered 'best practice'. This can differ between font foundries, UFO format flavours, and intended uses. This includes public specifications, but should support private ones.
- Provide both specifications for checking and recommendations for how specific errors can be fixed.
- Provide these in a public repostory that encourages contribution and refinement

This is not a tool, but rather a collection of data that can be used by other tools.

# Profiles

An individual profile contains a list of checks to perform, defined by the particular UFO key - one check per key. These checks are organized by the file in which the key is found. Each key has up to five different related elements. For purposes of discussion the profile is described here in YAML format, although the final format has yet to be determined. A partial hypothetical example:

```yaml
--- #UFO Checks profile
profilename:  UFO3_Common_Partial
maintainer:   SIL International
parent:       UFO3_Strict

fontinfo.plist:
    - key:    ascender
      req:    min
      type:   int

    - key:    guidelines
      req:    opt
      fix:    remove if empty

    - key:    italicAngle
      req:    opt
      type:   float
      test:   error 0 or missing, and styleName contains 'italic'
      fix:    remove if 0
      
    - key:    openTypeOS2Type
      req:    strict
      empty:  allow
      
    - key:    year
      req:    disc
      fix:    remove
...   
```

The profile begins with :

- __profilename__ *(required)* - The name used to refer to the profile in any UIs. May also be used by the __parent__ element to identify parent profile.
- __maintainer__ - The person or organization primarily responsible for maintaining the profile. They should be consulted in case of modifications.
- __parent__ - The profile that should be applied as the basis of this profile. Checks defined in the parent profile will be run as part of this one. If any keys in this profile are also in the parent profile, the check defined in this profile overrides the parent.

Then all the checks relevant to the profile are listed, grouped by the file in which the key appears, such as `fontinfo.plist`.

Each check is defined by the key and related elements. 

- __key__ (_required_) - The UFO spec name.
- __req__ (_required_) - Checks whether this key is present. There are five possible values, with corresponding reporting actions:
  - *min* - Key is minimally required, and font is likely to not build or function properly without it. If key is missing this will report an  __error__.
  - *strict* - Key is required in a strict interpretation of this profile spec.  If key is missing this will report a __warning__.
  - *opt* - Key is optional. Nothing is reported.
  - *disc* - Key is discouraged, as there may be a better or more proper place for the information, or it may cause problems. If the key is present this will report a __warning__.
  - *dep* - Key is deprecated or not allowed in this UFO flavour. If the key is present this will report an __error__.
- __empty__ - Checks whether the key has no specified value (is empty). By default, empty keys should not be present, and will be reported with a __warning__. This warning can be suppressed with either of two settings:
  - *allow* - Allow and do not report if there are empty keys.
  - *allowclosed* - Allow and do not report emoty keys, but only if represented by the self-closing format `<key />`
- __type__ - Checks whether the key value is in the specified type. If the type is different from that specified, it will be reported with a __warning__.
  - *int* - Integer
  - *float* - Float
- __test__ -  Checks the value of the key using the specified test. The test may be specified using calculations, reference to other keys, or specific values. It also needs to specify whether problems are reported as an __error__ or a __warning__. *How these tests should be specified is still uncertain, and suggestions would be appreciated. Should they be in human-readable form? A python function?* Also see discussion of Tests, below. Some examples of the types of tests needed:
  - (for *openTypeOS2TypoAscender*) Warning if key value is not equal to ascender
  - (for *italicAngle*) Error if 0 or missing, and styleName contains 'italic'
- __fix__ - This is not a check but rather an actual change to the UFO. This allows for direct changes to the UFO to meet a particular specification. Tools do not need to provide fixing capabilities. *How the fixes should be specified is also uncertain - suggestions very welcome.* Some examples of needed fixes:
  - (for *italicAngle*) Remove if 0
  - (for *guidelines*) Remove if empty
  - (for *openTypeOS2TypoAscender*) Set to ascender
  - (for *openTypeNameUniqueID*) Set to 'openTypeNameManufacturer: familyName styleName: currentyear' 

# Tests
An alternative way to specify complex tests might be to have a separate collection of tests in one or more files that can be referenced to from within profiles. There are some potential advantages to this:

- Would make profiles simpler, as they would contain only references to named tests. However it might also make it more cumbersome to see what the tests actually do. 
- Could provide common tests, such as 'Error if 0' that could be referred to in multiple places in multiple profiles.
- Could be specified as a module of python functions that can be directly used by tools.

# Fixes
Fixes could also be specified in separate files, with the same advantages as Tests.

# Notes
It is very useful and important to know the rationale behind various tests and fixes, and particularly if opinions differ. Notes would be held in one or more markdown files. Abbreviated example:

```markdown
# Notes for fontinfo.plist [UFO3](http://unifiedfontobject.org/versions/ufo3/fontinfo.plist/) [UFO2](http://unifiedfontobject.org/versions/ufo2/fontinfo.plist/)

## ascender
*(UFO)* Not required by spec but necessary for building fonts. Should be equal to other ascender metrics (*openTypeHheaAscender*, *openTypeOS2TypoAscender*, *openTypeOS2WinAscent*) unless there is a specific technical need to set them differently. See [Font Development Best Practices](http://silnrsi.github.io/FDBP/en-US/Line_Metrics.html).

## capHeight
*(UFO)* Not required, however ufo2ft seems to complain if it is not present. Having this set properly also makes a web font more useful in complex CSS layouts that may use this metrics for things like drop cap layout.

## year
*(UFO2)* Deprecated in UFO3, but the UFO spec says that authoring tools should preserve it if found.

However, this information is often misleading and may not be updated properly. Is it the year of the original design? Or of the last major revision? There are also more appropriate locations for the information, such as the copyright statement. For these reasons it may be better to not use or preserve this key in a UFO3 font.
```

# Other definitions

## Reporting levels

There are three different types of reported results:

- __Error__ - Check failed to meet a critically important  spec, and font needs to be fixed. This should be presented to the user as a serious problem.
- __Warning__ - Check revealed a problem that should be fixed, but is not likely to be a critical error. This most often may be the case for best practice or foundry-specific standards.
- __Info__ - No check failed, but test revealed information that may be useful for the user.

# Inspirations and references

[fontQA](http://www.fontqa.com/)

[Font Validator](https://github.com/HinTak/Font-Validator/)

[ufolint](https://github.com/source-foundry/ufolint)

[fontbakery](https://github.com/googlefonts/fontbakery)

[pysilfont](https://github.com/silnrsi/pysilfont)

