+++
title = 'Airflow'
date = 2025-02-01T10:06:46-08:00
draft = true
+++
## Ch01: Introduction
* Airflow allows defining DAGs as python code providing flexibiilty , make it easy to review and push changes in. It also allows for optional DAGs depending on certain conditions.
* after defining DAGs you can schedule it to run every day, hour.
* Airflow steps: it checks if the schedule for a DAG has passed, then extracts all tasks in task and schedules it for execution. next scheduler checks whether the dependencies have been satisfied to add the task to the execution queue. if dependencies are not satisfied then wait X sseconds before checking.
* revisit schedule interval and backfill for airflow.
* when not to use airflow 1) streaming pipelines as its primarily designed for batch pipelines 2) dynamic pipelines where tasks are added/removed between every pipeline run as the UI only shows tasks present in the latest DAG 3) it comes with operational cost so small teams might not be suited for this.
## Ch03: Scheduling in Airflow
* scheduling interval can be defined using cron syntax or predefined intervals like @daily, @weekly, @monthly. interval based schedulig is stateless i.e. when the given time is reached it creates an excecution of the DAG
* frequency based intervals depend on the previous run i.e. run every other day or every 3 days. previos run decides when the next run will be. this is difficult to do with cron and airflow it provides a functionality for this.
* for creating incremental data processing airflow provides date parameters. this is called execution_date, previous_execution_date and next_execution_date. execution date is same as dataset date. These define intervals for processing data e.g. audit date between execution_date and next_execution_date.
* airflow defines a start_date parameter which specifices when the DAG execution should start from. It also provides an end_date parameter to specifiy when the scheduling should stop.
* by default airflow will create all scheduled internals from start_date to execution_date. this is used for backfilling. set parameter catchup=False to disable this behavior.
* best practices
    * tasks should be atomic e.g. do not create data processing and sending notification in the same task as failure to send notification will fail the task even when data processing is complete. however keep related processes like authentication in the same task.
    * tasks should be idempotent i.e. failed tasks can be rerun without causing any side effects. this is commonly implemented by overwriting output files.
## Ch05: Defining Dependencies between tasks
* fan out dependencies task1 >> [list of fan out tasks]
* fan in dependencies [list of upstream tasks] >> task1
* branching can be done in tasks and within code. doing it in tasks make the branching more explicit, easier to understand and leverages capabilities of the orchestration tool like UI, logging etc. when branching with code its not easier to tell when a particular branch was executed unless there is logging. orcestration tool makes this understanding explicit.
* when branching and only one of the branch will be executed create a dummy task with trigger rule as none_failed. This will ensure downstream task rules are not updated due to this branching. upstream of the 2 branches create a new branch task that returns which of the 2 downstream tasks will be executed based on some logic.
* conditional tasks e.g. during backfill run ML training and deployment task only for latest execution date instead of all previous intervals.
* different types of trigger rules
    * all_success: default to run task when all parent tasks are successful
    * all_failed: when all parent tasks have failed. For error handling
    * all_done: when all parents are done regardless of their resulting state. for cleanup code
    * one_failed: quickly trigger error handling
    * one_success: quickly trigger notifications
    * none_failed: when no parents have failed but have completed successfully or skipped. when joining branches
    * none_skipped: when no parents are skipped regardless of their execution state
    * dummy: triggers regardless of the state of any upstream tasks. testing.
* share data between tasks using XComs like sharing model_id or file name published with downstream tasks.
## Ch05: Triggering Workflows
* apart from triggering based on schedule we can also trigger workflows and poll for certain conditions to be met e.g. waiting for a file to be available
* polling sensors occupy workers and if too many polling sensors are active there will be no workers left for running new tasks. this threshold is for both DAG and airflow system.
* lots of polling sensor tasks bulid up when previous run is not complete and subsequent ones are triggered adding more polling sensor.
* you can also set up dependency between DAGs by using a polling sensor which will poll for the completion of parent DAG. This is used to trigger a dependent DAG when one has completed. This allows us to break DAG into smaller sizes and also allow multiple DAG to take dependency on it.
* can trigger workflow from CLI as well.
## Ch07: Communicating with External Systems
* since airflow is an orchestration systems it commonly interacts with external systems specially in the cloud
* when connecting with external systems majority of work is configuring the tasks for working with external systems.
* a common ETL operation is to move data. We can create new operator of moving data between any 2 systems. It requires configuration to connect with the 2 systems and logic to move data between them.
* a common argument is the airflow workers should only be used to trigger tasks in external systems instead of actually executing tasks. This is a good mental model for separation of concerns.
## Ch10: Running tasks in containers
* having different type of tasks in airflow requires having all required dependencies to run them. this can sometimes lead to conflicts.
* also having different type of tasks means the learning curve for the DAG is higher.
* also merging orchestration and business logic together is not a recommended approach.
* these are addressed by using airflow to run tasks in containers instead of airflow workers.
* these containers also enable to test the business logic independently using mocks

## Testing
* 4 environments dev, test, acceptance, prod. dev has random sample data from prod. it only validates that the pipeline execute correctly, schema is as expected. test has specific records that validate the business logic. acceptance has prod data so it can be used as a validation layer for product owners or can be used to validate prod level features like distribution of data due to the change.
* data ingestion into data platform should be separate from data processing layer. This simplifies management and simplifies testing.
* 
