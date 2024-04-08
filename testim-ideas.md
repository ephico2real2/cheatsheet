stages:
  - e2e

testim:
  stage: e2e
  image: testim/cli:latest
  variables:
    TESTIM_TOKEN: <TESTIM_TOKEN>
    TESTIM_PROJECT: <TESTIM_PROJECT>
    TESTIM_LABEL: <TESTIM_LABEL>
    GRID_NAME: <GRID_NAME>
  script:
    - testim --token $TESTIM_TOKEN --project $TESTIM_PROJECT --label "$TESTIM_LABEL" --grid $GRID_NAME -r testim-report.xml
  artifacts:
    paths:
      - testim-report.xml
    reports:
      junit: testim-report.xml
