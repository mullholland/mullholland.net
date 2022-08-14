# Role - generate docs

The last thing we do, right after `molecule test` runs successfully and we are happy with the results and bevor we commit is `

```BASH
# execute the playbook from ansible-generator to create the documentation
/PATH/TO/REPOSITORY/ansible-generator/role-generator/generate_docs.yml
```

This generates/updates the following files:

- README.md
- .gitignore
- .ansible-lint
- .yamllint
- .editorconfig
- .github/workflows/galaxy.yml
- .github/workflows/molecule.yml
- .github/dependabot.yml
- .gitlab-ci.yml

Now we can commit and push out changes and the magic of CI should run the molecule tests on github and/or gitlab.
