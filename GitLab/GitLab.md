



# AUTH WITH GITLAB

local username and password (with 2FA)

ssh keys

Access Tokens           -       meant for a user
                                access to the API

Deploy token            -       meant for automation (e.g. GitOps via ArgoCD)
                                does have a username associated with it
                                no access to the API
                                allows to clone and pull a git repository (no push)
                                pull/push to container registry and package registry

External Providers
    - SAML
    - LDAP
    - OAUTH




# PIPELINES


RUNNER

- shared
- group
- project specific



PIPELINE

branch                  - runs when a commit is pushed
merge request           - runs when a merge request is created
    detached            - the pipeline ran only on the source branch of the merge commit
    merged results      - runs on the result of combining the source branch changes with target branch
                          (I read somewhere that the change has still not been merged into target as yet)
    merge train         - runs when merging multiple merge requests at the same time




GLOBAL DIRECTIVES

image
stages
cache               - specify a key and path
variables
before_script
after_script
workflow:
rules:
if:


JOB DIRECTIVES

stage
image
variables
cache
only
    - <branch-name>         - only run the job when this is the branch
    - merge_requests        - only run the job when a merge request has been created
except
    - schedules             - don't run the job if pipeline is kicked off from a    schedule
    - <branch-name>         - don't run the job for \<branch-name\> branch
script
artifacts
environment                 - think of an environment as a location of a deployment
    name: <env-name>
    url: <url>
    on_stop: <job-name>
when:                       
    - manual                - the job needs to be kicked off manually. 
                              (ALSO IT SEEMS THAT GITLAB CAN RUN THE JOB AFTER MERGE REQUEST WHEN SOURCE BRANCH IS BEING DELETED)
    - never

allow_failure: [true | false] - allow_failure will determine whether a pipeline should continue running when a job fails
                                true means continue running and false means stop the pipeline



VARIABLES

GIT_STRATEGY: none          - when this variable is defined in a job, GitLab will not clone the repo in the job run



GITLAB_INTERNAL_ACCOUNT_USER
GITLAB_INTERNAL_ACCOUNT_PASSWORD



CI_COMMIT_BRANCH
CI_COMMIT_REF_NAME




Where can a variable come from in a container runner

1. The image of the container
2. The Admin CI/CD settings
3. The Groups of the project CI/CD settings
4. The projects CI/CD settings
5. The .gitlab-ci.yml file variable declarations
6. Any script that gets involved in the pipeline could export a
variable



==============
Question

A job called 'stop review" marked to run as a manual job in
merge_requests type of pipeline was run when the source branch was
actually merged into target
The job basically stops an environment by running some commands
