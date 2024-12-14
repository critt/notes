### MobSF
- This tool seems pretty well-maintained
- They provide a command line utility called [mobsfscan](https://github.com/MobSF/mobsfscan) which seems pretty useful for automation, including in CI pipelines
- Pretty trivial to include these kinds of workflows in your CI pipelines for iOS and Android static analysis
###### Example GitHub Actions CI configuration file (`mobsfscan.yml`)
###### Note: I am currently using [this workflow in one of my repos](https://github.com/critt/Interp-Android/actions/workflows/mobsfscan.yml)
```yml
name: mobsfscan

on:
  push:
    branches: [master, develop]
  pull_request:
    branches: [master, develop]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4.2.2
      - uses: actions/setup-python@v5.3.0
        with:
          python-version: "3.12"
      - name: mobsfscan
        uses: MobSF/mobsfscan@main
        with:
          args: ". --json"
```

###### Example output
![[Screenshot 2024-12-13 at 5.31.45 PM.png]]

