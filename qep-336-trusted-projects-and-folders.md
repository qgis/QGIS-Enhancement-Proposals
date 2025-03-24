# QGIS Enhancement: Trusted Projects and Folders

**Date** 2025/01/07

**Author** Matthias Kuhn (@m-kuhn)

**Contact** matthias@opengis.ch

**Version** QGIS 3.X.X+

# Summary

When opening a QGIS project containing Python or other executable code (such as macros, feature actions , or expression functions), the user is currently prompted with a security warning. The options available are to "allow once," "always allow" (which generates a warning as it means completely disable protection for the current profile), "deny once," or "deny always." These options, while providing basic security measures, lack flexibility and sophistication in managing trusted environments, especially in enterprise settings.

## Proposed Solution


This QEP proposes the introduction of "Trusted Project" and "Trusted Folder" options to enhance security measures and user convenience in QGIS. This system would allow users to designate specific projects or entire folders as trusted, significantly reducing the frequency of security prompts for known safe content and streamlining workflow efficiency.

### Features and Benefits

- **Trusted Projects and Folders**: Users can mark individual projects or entire directories as trusted, which suppresses security warnings for Python code execution within those contexts.
  
- **Visual Studio Code-like Warning System**: A warning system, akin to that used by Visual Studio Code, will be implemented. This will inform users of the potential risks involved in executing Python code from untrusted sources, enhancing awareness and control over security settings.

- **INI File Configuration for Enterprises**: For enterprise environments, it will be possible to pre-configure trusted paths through a global INI file. This feature will allow system administrators to set up a secure, controlled environment by specifying trusted projects and folders, thereby simplifying configuration for end-users.

- **Improved Usability and Security**: By providing clearer options for managing trusted content, this proposal aims to improve the overall user experience. Users will have greater control over their security settings, allowing for a more organized and controlled access to projects containing executable code.

![image](https://github.com/qgis/QGIS-Enhancement-Proposals/assets/588407/b1cc5070-d82c-4b10-a763-7a9f634121bf)


### Implementation Considerations

- **User Interface**: The implementation will include UI enhancements to manage trusted projects and folders easily. This could involve a new section in the QGIS settings and project properties dialogues.

- **Backward Compatibility**: The introduction of this feature will be designed to be backward compatible, ensuring that existing projects and security settings are not adversely affected.

## Conclusion

The introduction of Trusted Projects and Folders in QGIS represents a significant step forward in enhancing the security and usability of the software. By allowing users and administrators to designate trusted environments, this feature will streamline workflows and provide a more intuitive and controlled approach to managing project security. This proposal aligns with the broader goals of improving QGIS's flexibility and robustness as a GIS platform, catering to both individual users and large-scale enterprise environments.
