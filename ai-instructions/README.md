The files in this directory are intended for use with a ChatGPT project.

1. Configure the project with the contents of `instructions.txt` as its instructions (only plain text is currently allowed).
2. Upload the other files to the project.
3. Use caution when updating files - new uploads do not replace existing versions. You must manually remove the old versions.

Start each conversation with the following prompt, substituting any other desired role for "generalist":
```text
Please initialize your behavior using the contents of roles.yaml, locating the generalist role, loading its definition file, and following all of the instructions in that file.
```

You may ask the assistant to switch roles and it should load the appropriate rules and modify its behavior accordingly.

Don't forget to rename the chat or it will linger forever with a title about loading the role.