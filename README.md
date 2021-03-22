# Package Aware circleci-orb

A [CircleCI Orb](https://circleci.com/docs/2.0/orb-intro/) for using [PackageAware](https://packageaware.io) to check for
vulnerabilities in your projects.

You can use the Orb as follows:

```yaml
version: 2.1

orbs:
  packageaware: packageaware/packageaware@dev:first

#
# The Workflow is the example of how a user would integrate with the PA ORB
#
#
# The Workflow is the example of how a user would integrate with the PA ORB
#
examples:
#  workflows:
 #   main:
  #    jobs:
        # - packageaware-orb/init

  async-workflow-common:
    # In this workflow, we run a standard/common asynchronous scan. It will execute
    # the "packageaware/analysis_async_init" job first, which initiates the scanning process at PackageAware.
    # You are free to add additional jobs after the "packageaware/analysis_async_init" step.
    # You must terminate the workflow with a call to "packageaware/analysis_async_result" if the result is important
    # to you and you wish to have your build impacted by the presence of analysis result failures.
    usage:
      version: 2.1
      workflows:
        main:
          jobs:
            - packageaware/analysis_async_init:
                # Turn off debugging (set to false) once you're comfortable with using the orb and no longer need the additional detail
                fs_debug: true
                # Change the project name to something specific to your project
                project_name: "My-Project-Name"

            # RUN YOUR OTHER JOBS HERE

            - packageaware/analysis_async_result:
                # Turn off debugging (set to false) once you're comfortable with using the orb and no longer need the additional detail
                fs_debug: true
                # Change the project name to something specific to your project
                project_name: "My-Project-Name"

                requires:
                 - packageaware/analysis_async_init

  run-and-wait-workflow-common:
    # In this worfklow, we run a standard/common synnchronous scan. It will start and fully complete (or time-out)
    # before control is given to addition steps in the workflow.
    usage:
      version: 2.1
      workflows:
        main:
          jobs:
            - packageaware/analysis_run_and_wait:
                # Turn off debugging (set to false) once you're comfortable with using the orb and no longer need the additional detail
                fs_debug: true

                # Change the project name to something specific to your project
                project_name: "My-Project-Name"

  async-workflow-initiate-only:
    # In this workflow, we run a asynchronous scan and we do not wait for the analysis result.
    # PackageAware will run the analysis as a result of this workflow, but the result will not impact your workflow
    # in any way as the workflow will not wait for the result.
    usage:
      version: 2.1
      workflows:
        main:
          jobs:
            - packageaware/analysis_async_init:
                # Turn off debugging (set to false) once you're comfortable with using the orb and no longer need the additional detail
                fs_debug: true
                # Change the project name to something specific to your project
                project_name: "My-Project-Name"

  run-and-wait-workflow-exclude-files-and-directories:
    # In this workflow, we inform the ORB that specific directories (and therefore subdirectories of those directories)
    # as well as specific individual files should be avoided/ignored when the ORB finds dependency files that match
    # the given criteria. When dependency files are ignored, they are not sent to the PackageAware service and
    # are therefore not scanned by the PackageAware service.
    usage:
      version: 2.1
      workflows:
        main:
          jobs:
            - packageaware/analysis_run_and_wait:
                # Completely exclude sub_folder/ and sub_folder_2/ and their contents.
                directories_to_exclude: "sub_folder/,sub_folder_2/"
                # Specifically exclude files:
                #   sub_folder_3/file_1.py
                #   sub_folder_3/file_2.py
                #   sub_folder_4/file_1.py
                files_to_exclude: "sub_folder_3/file_1.py,sub_folder_3/file_2.py,sub_folder_4/file_1.py"
                # Turn off debugging (set to false) once you're comfortable with using the orb and no longer need the additional detail
                fs_debug: true
                # Change the project name to something specific to your project
                project_name: "My-Project-Name"


  run-and-wait-workflow-long-processing-times:
    # In this workflow, we increase the ORB "max wait time" in order to make a best effort to avoid timing out.
    # The default timeout is 300 seconds, but in certain circumstances the default is not large enough:
    #        If you have an exceedingly large number of dependency files for PackageAware to process.
    # and/or If you have dependency files with an exceedingly long list of dependencies
    usage:
      version: 2.1
      workflows:
        main:
          jobs:
            - packageaware/analysis_run_and_wait:
                # The analysis_result_max_wait controls how long the process will wait for an analysis result before
                # it quits (in seconds.) 1200 seconds, below, is an arbitrarily large number. Please tune to your
                # system's needs.
                analysis_result_max_wait: 1200  # default is 300 seconds

                # Turn off debugging (set to false) once you're comfortable with using the orb and no longer need the additional detail
                fs_debug: true

                # Change the project name to something specific to your project
                project_name: "My-Project-Name"


  run-and-wait-workflow-continue-on-failure:
    # In this workflow, we inform the ORB that runtime failures or analysis result failures should be ignored
    # and the build process should continue in the event that an error is encountered.
    usage:
      version: 2.1
      workflows:
        main:
          jobs:
            - packageaware/analysis_run_and_wait:
                on_failure: "continue_on_failure"

                # Turn off debugging (set to false) once you're comfortable with using the orb and no longer need the additional detail
                fs_debug: true

                # Change the project name to something specific to your project
                project_name: "My-Project-Name"

        
```

The Package Aware Action has properties which are passed to the action using `with`.

| Property | Default | Description |
| --- | --- | --- |
| on_failure | "fail_the_build"  | Flag indicating whether or not to return an error code if errors are found in the Package Aware script or Package Aware analysis.
| directories_to_exclude | ""  | List (comma separated) of directories (relative to ./) to exclude from the search for manifest files. Example - Correct: bin/start/ ... Example - Incorrect: ./bin/start/ ... Example - Incorrect: /bin/start/'|
| files_to_exclude | "" | List (comma separated) of files (relative to ./) to exclude from the search for manifest files. Example - Correct: bin/start/manifest.txt ... Example - Incorrect: ./bin/start/manifest.txt ... Example - Incorrect: /bin/start/manifest.txt' |
| analysis_result_max_wait | 300 | Maximum seconds to wait for Analysis Result before exiting with error. |
| analysis_result_polling_interval | 10 | Polling interval (in seconds) for analysis result completion (success/failure.). Min 10. |
| fs_debug | false | Enables printing of debug statements from the Orb |

The Package Aware Action has environment variables which are passed to the action using `env`. These environment variables are stored as project `environment variables` and are required for the action to operate.

| Property | Description |
| --- | --- |
| PACKAGE_AWARE_PROJECT_NAME | A custom project name that will present itself as a collection of test results within your packageaware.io dashboard. |
| PACKAGE_AWARE_BASE_URI | The API BASE URI provided to you when subscribing to Package Aware services. |
| PACKAGE_AWARE_ROOT_CODE_PATH | The relative path from the workspace to search for manifest files to analyze. |
| PACKAGE_AWARE_CLIENT_ID | Provided to you when subscribing to Package Aware services. |
| PACKAGE_AWARE_API_KEY | Provided to you when subscribing to Package Aware services. |


## EXAMPLE: Asynchronous scan that contains other CI logic between the two Package Aware jobs:

```yaml
version: 2.1

orbs:
  packageaware: packageaware/packageaware@dev:first

workflows:
  main:
    jobs:

      # NOTE: YOUR OTHER JOBS GO HERE

      - packageaware/analysis_async_init:
          on_failure: "fail_the_build"
          directories_to_exclude: ""
          files_to_exclude: ""
          working_directory: $CIRCLE_WORKING_DIRECTORY
          analysis_result_max_wait: 300
          analysis_result_polling_interval: 10
          fs_debug: true

      # NOTE: YOUR OTHER JOBS GO HERE

      - packageaware/analysis_async_result:
          on_failure: "fail_the_build"
          directories_to_exclude: ""
          files_to_exclude: ""
          working_directory: $CIRCLE_WORKING_DIRECTORY
          analysis_result_max_wait: 300
          analysis_result_polling_interval: 1
          fs_debug: true

          # NOTE: RUNNING ASYNCHRONOUSLY WILL REQUIRE A DEPENDENCY TO BE ESTABLISHED AGAINST THE "analysis_async_init" JOB
          requires:
           - packageaware/analysis_async_init

        # NOTE: YOUR OTHER JOBS GO HERE
        
```
### ENVIRONMENT VARIABLES FOR THE ABOVE EXAMPLE
| Property | Value |
| --- | --- |
| PACKAGE_AWARE_PROJECT_NAME | "My Project Name" |
| PACKAGE_AWARE_BASE_URI | "https://api.packageaware.io/api/" |
| PACKAGE_AWARE_ROOT_CODE_PATH | "./" |
| PACKAGE_AWARE_CLIENT_ID | [redacted] |
| PACKAGE_AWARE_API_KEY | [redacted] |

## EXAMPLE: Synchronous scan that continues running until analysis complete or timeout reached:

```yaml
version: 2.1

orbs:
  packageaware: packageaware/packageaware@dev:first

workflows:
  main:
    jobs:

      # NOTE: YOUR OTHER JOBS GO HERE

      - packageaware/analysis_run_and_wait:
          on_failure: "fail_the_build"
          directories_to_exclude: ""
          files_to_exclude: ""
          working_directory: $CIRCLE_WORKING_DIRECTORY
          analysis_result_max_wait: 300
          analysis_result_polling_interval: 10
          fs_debug: true

      # NOTE: YOUR OTHER JOBS GO HERE
      
```
### ENVIRONMENT VARIABLES FOR THE ABOVE EXAMPLE
| Property | Value |
| --- | --- |
| PACKAGE_AWARE_PROJECT_NAME | "My Project Name" |
| PACKAGE_AWARE_BASE_URI | "https://api.packageaware.io/api/" |
| PACKAGE_AWARE_ROOT_CODE_PATH | "./" |
| PACKAGE_AWARE_CLIENT_ID | [redacted] |
| PACKAGE_AWARE_API_KEY | [redacted] |


## EXAMPLE: "Fire and Forget" scan that runs and the analysis result is inconsequential to the CI build.

```yaml
version: 2.1

orbs:
  packageaware: packageaware/packageaware@dev:first

workflows:
  main:
    jobs:

      # NOTE: YOUR OTHER JOBS GO HERE

      - packageaware/analysis_async_init:
          on_failure: "fail_the_build"
          directories_to_exclude: ""
          files_to_exclude: ""
          working_directory: $CIRCLE_WORKING_DIRECTORY
          analysis_result_max_wait: 300
          analysis_result_polling_interval: 10
          fs_debug: true

      # NOTE: YOUR OTHER JOBS GO HERE
      
```
### ENVIRONMENT VARIABLES FOR THE ABOVE EXAMPLE
| Property | Value |
| --- | --- |
| PACKAGE_AWARE_PROJECT_NAME | "My Project Name" |
| PACKAGE_AWARE_BASE_URI | "https://api.packageaware.io/api/" |
| PACKAGE_AWARE_ROOT_CODE_PATH | "./" |
| PACKAGE_AWARE_CLIENT_ID | [redacted] |
| PACKAGE_AWARE_API_KEY | [redacted] |
