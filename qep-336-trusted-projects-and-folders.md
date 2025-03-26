# QGIS Enhancement: Trusted Projects and Folders

**Date** 2025/01/07

**Author** Matthias Kuhn (@m-kuhn)

**Contact** matthias@opengis.ch

**Version** QGIS 3.X.X+

# Summary

When opening a QGIS project containing Python or other executable code (such as macros, feature actions , or expression functions), the user is currently prompted with a security warning in the message bar.

> **Seurity warning:** Python macros cannot currently be run. [[Enable Macros]]

This has a few problems:

 - It does a poor job in informing the user about the risks
 - It offers the risky option as the only choice (you need to know you can click away the bar in other ways)
 - A user with only one or a few projects with acceptable code use require the user to repeatedly allow for every load, effectively training users to just "click away the usual message"

In the options dialog there are also options to "allow always", "never allow", "allow for current session".

These options, while providing basic security measures, lack flexibility and sophistication in managing trusted environments, especially in enterprise settings.

## Proposed Solution

This QEP proposes the introduction of "Trusted Project" and "Trusted Folder" options to enhance security measures and user convenience in QGIS. This system would allow users to designate specific projects or entire folders as trusted, significantly reducing the frequency of security prompts for known safe content and streamlining workflow efficiency.

### Features and Benefits

- **Trusted Projects and Folders**: Users can mark individual projects or entire directories as trusted, which suppresses security warnings for Python code execution within those contexts.
  
- **Visual Studio Code-like Warning System**: A warning system, akin to that used by Visual Studio Code, will be implemented. This will inform users of the potential risks involved in executing Python code from untrusted sources, enhancing awareness and control over security settings.

- **INI File Configuration for Enterprises**: For enterprise environments, it will be possible to pre-configure trusted paths through a global INI file. This feature will allow system administrators to set up a secure, controlled environment by specifying trusted projects and folders, thereby simplifying configuration for end-users.

- **Improved Usability and Security**: By providing clearer options for managing trusted content, this proposal aims to improve the overall user experience. Users will have greater control over their security settings, allowing for a more organized and controlled access to projects containing executable code.

![image](https://github.com/qgis/QGIS-Enhancement-Proposals/assets/588407/b1cc5070-d82c-4b10-a763-7a9f634121bf)


### Implementation Considerations

- **User Interface**: The implementation will include UI enhancements to manage trusted projects and folders easily. When opening a project with code inside the user interface will propose to add the current 
  project or parent folder. In the QGIS options dialog, it will be possible to manage the trusted folders.

- **Backward Compatibility**: The introduction of this feature will be designed to be backward compatible, ensuring that existing projects and security settings are not adversely affected.

- **Storage**: The list of trusted folders and projects will be stored in QSettings (the profile ini file)

- **Introducing code during configuration**: If a project has been loaded without any code and at some point code is added during configuration, we will try to show this dialog as soon after as possible. E.g. when   applying the project properties.

- **Project location**: The project location to check for is the physical project filename, NOT the project home which is user configurable.

- **Cryptography**: This is deliberately not part of this proposal, as it enhances complexity not only on code side but also on user side by requiring either more setup (e.g. private/public keys, identity management) whereas this focuses on a low-threshold and accessible implementation

### API Considerations

The core of the new API will be the following

```cpp
enum EmbeddedCodeType
{
  Macro,
  ExpressionFunction,
  Action,
  FormInitCode
};

struct EmbeddedCode
{
  EmbeddedCodeType type;
  QString code; // the actual code
  // EmbeddedCodeLanguage language; // possibly added in the future
}

QList<EmbeddedCode> QgsProject::embeddedCode() const; // read-only. Traverses project settings like macros, expression functions etc.
```

### Future enhancements

In the future, a few extensions are thinkable:

- A listing with all the code in the current project can be shown for inspection and more informed decision making. The design for this is already around (see _API Considerations_ above)

- Trusting only specific project versions, identified by a hash. This is not part of phase 1, as we expect this also to come with the unwanted effect of an increased warning fatigue.

## Conclusion

The introduction of Trusted Projects and Folders in QGIS represents a significant step forward in enhancing the security and usability of the software. By allowing users and administrators to designate trusted environments, this feature will streamline workflows and provide a more intuitive and controlled approach to managing project security. This proposal aligns with the broader goals of improving QGIS's flexibility and robustness as a GIS platform, catering to both individual users and large-scale enterprise environments.
