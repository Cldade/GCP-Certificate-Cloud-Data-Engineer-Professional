# Task 1: Lab preparations

## Clone the training github repository

In the training-vm SSH terminal enter the following command:

`git clone https://github.com/GoogleCloudPlatform/training-data-analyst`

# Task 2: Identify map and reduce operations

- Return to the training-vm SSH terminal and navigate to the directory /training-data-analyst/courses/data_analysis/lab2/python and view the file is_popular.py with Nano. **Do not make any changes to the code**. Press Ctrl+X to exit Nano.

`cd ~/training-data-analyst/courses/data_analysis/lab2/python
nano is_popular.py`

Can you answer these questions about the file is_popular.py?

- What custom arguments are defined?

The following custom arguments are defined using the argparse.ArgumentParser:

1. --output_prefix: This argument specifies the output prefix for the result files. It has a default value of '/tmp/output', but it can be overridden by providing a different value when running the script.
2. --input: This argument specifies the input directory or file pattern for the Java files to be processed. It has a default value of '../javahelp/src/main/java/com/google/cloud/training/dataanalyst/jav>', but it can be modified to point to a different directory or file pattern.

These custom arguments allow the user to specify the input and output locations flexibly when running the script. By providing different values for these arguments, the user can process different sets of Java files and store the results in different output directories.

- What is the default output prefix?

the default output prefix is '/tmp/output'

- How is the variable output_prefix in main() set?

The variable output_prefix in the main() function is set based on the value provided for the --output_prefix argument when running the script.

- How are the pipeline arguments such as --runner set?

After parsing the command-line arguments using parser.parse_known_args(), the parsed arguments are stored in the options variable. The pipeline_args variable contains the remaining arguments that were not recognized by the argparse.ArgumentParser.

The pipeline_args variable, which includes the --runner argument and any other unrecognized arguments, is then passed as the argv argument to the beam.Pipeline() constructor when creating the pipeline (p).

By specifying the --runner argument when executing the script, you can set the desired execution runner for the Apache Beam pipeline. For example, you could provide --runner DataflowRunner to execute the pipeline on Google Cloud Dataflow, or --runner DirectRunner to execute the pipeline locally. The specific behavior and available runners depend on the Apache Beam implementation and the execution environment.

- What are the key steps in the pipeline?

The key steps in the pipeline can be summarized as follows:

1. Reading Java files.
2. Filtering lines with the "import" keyword.
3. Extracting and splitting package names.
4. Combining package usage.
5. Finding the top 5 packages.
6. Writing the results to an output file.

In summary, the pipeline reads Java files, extracts imported packages, counts their usage, and identifies the top 5 most used packages. The results are then written to an output file.

- Which of these steps are aggregations?

Step 4.

# Task 3. Execute the pipeline

In the training-vm SSH terminal, run the pipeline locally:

`python3 ./is_popular.py`

Identify the output file. It should be output<suffix> and could be a sharded file:

`ls -al /tmp`

Examine the output file, replacing '-*' with the appropriate suffix:

`cat /tmp/output-*`

OUTPUT: [('org', 45), ('org.apache', 44), ('org.apache.beam', 44), ('org.apache.beam.sdk', 43), ('org.apache.beam.sdk.transforms', 16)]

# Task 4. Use command line parameters

In the training-vm SSH terminal, change the output prefix from the default value:

`python3 ./is_popular.py --output_prefix=/tmp/myoutput`

What will be the name of the new file that is written out?

Note that we now have a new file in the /tmp directory:

`ls -lrt /tmp/myoutput*`



