# Notebook to test api4jenkins - create job, update job

## To run:

1. Clone the repository
2. `cd evalApi4Jenkins/`
3. Create and activate virtualenv
4. `pip install -r requirements.txt`
5. Open Jupyter notebook server `jupyter notebook`
6. Open and run `test.ipynb`


## Notes:

1. For creating custom jobs through code, we first created a manual job, fetch a job using `manualJob = j.get_job('Python-Application')` and fetch its xml using `xml = manualJob.configure()`. Then we fortmatted the xml, changing the description and git url. Then finally we created a similar Pipeline job through code using createPipeline() function, passing a new name and the formatted xml string.

2. 500: Internal server error was faced in creating a PipelineJob using the createPipeline() function, due to exact formatting of the XML string. There should be no preceding spaces in an XML string.

3. How I figured out the issue mentioned above ?
I checked Jenkins system logs from `http://localhost:8080/log/all` Or Manage Jenkins --> System Logs --> All logs in WebApp. Finally solved it following the issue, referenced from [StackOverFlow](https://stackoverflow.com/questions/19889132/error-the-processing-instruction-target-matching-xxmmll-is-not-allowed)

4. To further create different types of jobs in Jenkins, we will first create manual Jobs, fetch their XML, identify differences in XML and create small snippets for different jobs like Pipleline, Mulitbranch pipeline, folder, etc. Furthermore depending upon the job we are creating we will need to parse and create the custom XML using these snippets and finally create the job. We may need to define custom createJobs() functions for different jobs we are creating.