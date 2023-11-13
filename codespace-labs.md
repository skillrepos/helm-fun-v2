# Helm Fundamentals
## Deploying, upgrading, and rolling back applications in Kubernetes

## Revision 1.1 - 11/12/23

**Startup IF NOT ALREADY DONE!**
```
alias k=kubectl
minikube start
```

**NOTE: To copy and paste in the codespace, you may need to use keyboard commands - CTRL-C and CTRL-V.**

**Lab 1: Repos and Charts**

**Purpose: In this lab, we'll start working with Helm by adding a repo and pulling down a Helm chart, installing it, and then updating the release.**

1.	First, let's verify what version of Helm we have installed (should be 3.0+)

```
helm version
```

2.	Now let's add a couple of public chart repositories

```
helm repo add stable https://charts.helm.sh/stable
helm repo add cm https://chartmuseum.github.io/charts
```

3.	Verify that the charts were installed by listing the repos Helm knows about.

```
helm repo list
```

4.	You should see the stable and cm repo listed.  Now let's see what charts are available for download from these.

```
helm search repo stable
```
(What do you notice about all the charts here?)
```
helm search repo cm
```

5.	There are a lot of charts available.  We're going to setup chartmuseum - a repository for Helm charts - so we'll have our own local repository.  First, let's find the available ChartMuseum versions.

```
helm search repo chartmuseum
```

The output should show you the currently available versions of the chart across repos.


6.	Let's get some general information about the most recent version of the chart.

```
helm show chart cm/chartmuseum
```

7.	Verify that no releases are currently installed on the cluster.

```
helm list -A
```

8.	Now let's install chartmuseum locally.  Recall that we need to give it a name when we install.

```
helm install local-chartmuseum cm/chartmuseum --version 2.15.0
```

9.	Verify that the chart installation created a release in the cluster.  What revision is it?

```
helm list
```

10.	Notice the output you got when you did the helm install in step 8.  This is telling us that in order to access this in the browser, we need to forward the port from within the cluster out.  There are a couple of ways to do this, but one of the simplest is just to port-forward the service. You can see the service running and then run the second command below to do the port-forward.

```
k get svc
k port-forward svc/local-chartmuseum-chartmuseum 8080:8080
```

11.  After executing this command, you'll see a popup in the lower right with a button to click on to see Chartmuseum running. (If the dialog goes away, you can click on the *PORTS* tab in the top "tab" line of the terminal, find the row with "8080" in the *Port* column, and click on that to open it up in a browser.)	

![Opening app via dialog](./images/helmfun5.png?raw=true "Opening app via dialog")
   

12. After this, you should get a simple browser that opens up as a pane in the editor.

![Opening cm in browser](./images/helmfun6.png?raw=true "Opening cm in browser")

<p align="center">
**[END OF LAB]**
</p>
 

**Lab 2:  Changing values**

**Purpose: In this lab, you’ll get to see how we can change values and upgrade releases through Helm, as well as learn some more Helm commands.**

1. In order to do the following steps, we'll need to open a second terminal. We can do that by splitting this one. Either right-click and select Split Terminal or click on the two-panel icon near the trash can. See screenshot.

![Opening split screen](./images/helmfun7.png?raw=true "Opening split screen")

2. The service that runs in the Kubernetes cluster for ChartMuseum is defaulted to be of type "ClusterIP" - mainly intended for traffic internal to the cluster. To see that, in the second terminal window, set the alias, and run the command to get the service info for the namespace where ChartMuseum is running.  
```
alias k=kubectl
k get svc
```
Notice the output indicating the type of service is ClusterIP.

3. Let's change the chartmuseum service to be of type NodePort instead of ClusterIP.  That will let us have a node in the range 30000-32767 that can be used for access.  Let's first see what values we have that can be changed in our chartmuseum chart 

```
helm show values cm/chartmuseum
```

4. Lots of information there, but we'd like to change only the chart type and assign a node port.  Let's look for the information around the text "ClusterIP"

```
helm show values cm/chartmuseum | grep -n14 ClusterIP
```

5. Notice under the output, in the "service" section, we have the "type" set to ClusterIP and an empty setting for "nodePort".

6. Remind yourself of what release you currently have out there.

```
helm list
```

7. Go ahead and stop the port-forwarding that was running in the other window (via Ctrl-C).

```
Ctrl-C
```
  
8. We want to upgrade the service.type and service.nodePort values in our Helm release.  How do we do that on the command line?  Take a look at the first part of the help for helm upgrade.

```
helm upgrade --help | head
```

9. Notice the last line about being able to use the --set option to override values from the command line.  We'll run an upgrade and try that - explicitly setting the service.type to NodePort and the node port itself to 31000. (You may want to copy and paste this one.)

```
helm upgrade local-chartmuseum cm/chartmuseum --set env.open.DISABLE_API=false --set service.type=NodePort --set service.nodePort=31000
```

10. Take a look at the release you have out there now, its status and its history

```
helm list
helm status local-chartmuseum
helm history local-chartmuseum
```

11. Verify that the type of port has been changed for the running version.  

```
k get svc
```

You should see it listed as a NodePort now with 31000 as the exposed port.

12. You need to port-forward again. For convenience, there's also a simple script you can run to do this, passing in the new port.

```
extra/cm-port.sh 31000 &
```
OR
```
k port-forward svc/local-chartmuseum 31000:8080 &
```

13. After executing this command, you'll see a popup in the lower right with a button to click on to see Chartmuseum running. (If the dialog goes away, you can click on the *PORTS* tab in the top "tab" line of the terminal, find the row with "8080" in the *Port* column, and click on that to open it up in a browser.)	

![Opening app via dialog](./images/helmfun8.png?raw=true "Opening app via dialog")
   

14. After this, you should get a browser tab with Chart Museum running.

![Opening cm in browser](./images/helmfun6.png?raw=true "Opening cm in browser")

15. Now we'll add your chartmuseum repo to your list of repos for Helm and verify it's there. **In your other terminal session:**

```
helm repo add local  http://localhost:31000
helm repo list
```
<p align="center">
**[END OF LAB]**
</p>

**Lab 3: Creating a Helm Chart**

**Purpose: In this lab, we'll create a simple Helm chart and add it to our new repository**

1. Let's use Helm to create a simple, default chart one that will spin up an nginx deployment.

```
helm create sample-chart
```

2. Let's see what Helm created in terms of the structure of files and directories.  

```
tree sample-chart
```

3. Change into the *sample-chart* directory

```
cd sample-chart
```


4. Now take a look at some of the main files in the new chart. You can select each of the files to look at ones you're interested in. Switch back to the labs doc when done.
   
Select [**sample-chart/Chart.yaml**](./sample-chart/Chart.yaml) to open it.

Select [**sample-chart/values.yaml**](./sample-chart/values.yaml) to open it.

Select [**sample-chart/templates/deployment.yaml**](./sample-chart/templates/deployment.yaml) to open it.

Select [**sample-chart/templates/service.yaml**](./sample-chart/templates/service.yaml) to open it.


5.  Take a look at how Helm would render files in this chart.

```
helm template --debug . | head -n 50
```

6. Go ahead and install the chart.

```
helm install sample .
```

You will see some output like this:
```
NAME: sample
LAST DEPLOYED: <date/time>
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
  Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=sample-chart,app.kubernetes.io/instance=sample" -o jsonpath="{.items[0].metadata.name}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:80
```

7.  If you do a helm list command, you'll see that the chart was deployed in the "default" namespace (alongside our chart-museum one) and note that it is release version 1. You can also see the Kubernetes objects that were deployed in the default namespace for this.

```
helm list
k get all | grep sample
```

8. Take a look at the rendered templates that got deployed into the cluster.

```
helm get manifest sample | head -n 50
```

This should look very similar to the output from the template command that was issued earlier.

9. You may have noticed earlier that this chart had a test built into it.  Let's run the test now.  Note the output and also note the pods that are there afterwards.  Then take a look at the definition of the test afterward and see if you can understand how it all ties together.

```
helm test sample
k get pods
cat templates/tests/test-connection.yaml
```
 
10. We're done with this release now, so we can delete it.

```
helm delete sample
```
11. Look at the objects in the default namespace to see that the sample ones were removed.  You may see a leftover test pod that did not get removed.  If so, use the second command below to remove it.

```
k get all | grep sample
k delete pod/<sample-pod-name>
```

<p align="center">
**[END OF LAB]**
</p>

**Lab 4: Charts and Dependencies**
**Purpose:  In this lab, we’ll deploy the chart for our sample webapp, and then see how to add another chart as a dependency for its database.**

1. Go to the main directory for our helm work and switch to the quay.io branch.  You can optionally look at any of the files you're interested in.

```
cd /workspaces/helm-fun-v2
git stash
git checkout quay.io
cat <files of interest>
```

2. Create a namespace for the app to run in and then deploy it via Helm

```
k create ns roar
cd roar-web
helm install -n roar roar .
```

Afterwards you should see a set of output like the following:
```
NAME: roar
LAST DEPLOYED: Mon Jun 20 21:21:47 2022
NAMESPACE: roar
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

3. Take a look at the release that has been installed in the namespace. 

```
helm list -n roar
```

You should see output like the following:

```
NAME    NAMESPACE       REVISION        UPDATED                                STATUS   CHART           APP VERSION
roar    roar            1               2022-06-20 21:21:47.8781047 -0400 EDT  deployed roar-web-0.1.0
```
 
Now look at the resources that are installed in the namespace.

```
k get all -n roar
```

4. Find the nodeport where the app is running.  

```
k get svc -n roar   
```

Look for the NodePort setting in the service output (should be a number > 30000 after "8089:")

5. Forward the port from step 4 to 8080 on the host machine.  You can use the provided script below to do this.

```
../extra/roar-port.sh roar <nodeport from step 4>   
```

6. After executing this command, you'll see a popup in the lower right with a button to click on to see the application running. (If the dialog goes away, you can click on the *PORTS* tab in the top "tab" line of the terminal, find the row with the node port in the *Port* column, and click on that to open it up in a browser.)	

![Opening app via dialog](./images/helmfun9.png?raw=true "Opening app via dialog")
   

7. After this, you should get a browser tab with the web server running. To see the application running, add "/roar/" to the end of the URL.  **Make sure to include the trailing slash in /roar/**

![Opening app in browser](./images/helmfun10.png?raw=true "Opening app in browser")


8. Notice that we see our webapp, but there is no data in the table.  This is because we don't have our database deployed and being pulled in. We have a database chart and resources on our local system. Let's see how to get it pulled in as a dependency.  First, change into the directory for the database pieces.  Again, you can look at any files of interest in there.

```
cd /workspaces/helm-fun-v2/roar-db
cat <files of interest> or open them up via clicking on them in the file list
```

9. We want to package this up so it can be used as a dependency for our web chart. Go ahead and run the command to package it up. You should be in the roar-db subdirectory still.

```
helm package . 
```

For output, you should see something like this:
```
Successfully packaged chart and saved it to: /home/diyuser3/helm-ws/roar-db/roar-db-0.1.0.tgz
```

10. Upload the newly created package to our ChartMuseum instance. (This command is also in the file commands.txt in the extra subdirectory -   ~/helm-ws/extra/commands.txt )

```
curl --data-binary "@roar-db-0.1.0.tgz" http://localhost:31000/api/charts
```
When done, you should see output like this:

{"saved":true}

11. Now that we have this package stored in our chart repository, we can add it as a dependency into our webapp's chart so it will have data to display.  To do that, we add it to the Chart.yaml file.  We have a before and after version of the file. Diff the two files with the code diff tool to see the differences.

```
cd /workspaces/helm-fun-v2/roar-web
code -d ../extra/Chart.yaml Chart.yaml
```

12. Now we'll update our Chart.yaml file with the needed changes.  To save trying to get the yaml all correct in a regular editor, we’ll just use the diff tool’s merging ability. In the diff window, between the two files, click the arrow that points right to replace the code in our roar-web/Chart.yaml file with the new code from extra/Chart.yaml.  (In the figure below, this is the arrow that is circled and labelled "1".) After that, the files should be identical and you can close the diff window (circled "2" in the figure below).

![Diff and merge in code](./images/helmfun11.png?raw=true "Diffing and merging for dependent chart")

	
13. Now, update your dependencies.

```
helm dep up
```

You should see output like this as Helm gets your new dependency:
```
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "local" chart repository
...Successfully got an update from the "cm" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈
Saving 1 charts
Downloading roar-db from repo http://localhost:31000
Deleting outdated charts
```

14.  If you look at the directory structure now, you'll have a new "charts" directory where the tgz file will be. With our new dependency in place, let's remind ourselves what is out there now and then go ahead and upgrade our webapp release.

```
ls charts
helm list -n roar
k get all -n roar
helm upgrade -n roar roar  .  --recreate-pods (ignore the warning here)
```

15. Take a look at what we have out there now for this release.

```
helm list -n roar
k get all -n roar
```

You should see the various database pieces in the cluster now.

16.  The roar-port.sh command will likely have stopped.  Run it again to pick up the new pod.

```
../extra/roar-port.sh roar <nodeport from step 4>   
```

17. Finally, refresh the webapp in your browser and you should see data being displayed.

![App with data](./images/helmfun18.png?raw=true "App with data")

<p align="center">
**[END OF LAB]**
</p> 


**Lab 5: Templating**

**Purpose:  In this lab, we'll see how to add templating to a manifest file and also another way to specify values and setup dependencies.**

1. Let's suppose we want to create a helm deployment passing in a test database. We're going to change our dependency to be an actual copy of the chart so we can work with it more easily. 

```
cd /workspaces/helm-fun-v2/roar-web (if not already there)
rm -rf charts/* 
cp -R ../roar-db  charts/
```

2. Take a look at the deployment.yaml in the sub-chart.  Notice that the image name is hardcoded and not templated.

Select [**roar-db/templates/deployment.yaml**](./roar-db/templates/deployment.yaml) to open it.

Here's the relevant portion:

```
   - name: {{ .Chart.Name }}
     image: quay.io/bclaster/roar-db-image:v1
     imagePullPolicy: Always
     ports:
```

3. We want to edit that file and change that part to a templated format. To do that, we add it to the *deployment.yaml file*.  We have a before and after version of the file. Diff the two files with the code diff tool to see the differences.

```
code -d ../extra/deployment5-1.yaml charts/roar-db/templates/deployment.yaml
```

4. Now we'll update our Chart.yaml file with the needed changes.  To save trying to get the yaml all correct in a regular editor, we’ll just use the diff tool’s merging ability. In the diff window, between the two files, click the arrow that points right to replace the code in our roar-db/templates/deployment.yaml file with the new code from extra/deployment5-1.yaml.  (In the figure below, this is the arrow that is circled and labelled "1".) After that, the files should be identical and you can close the diff window (circled "2" in the figure below).

![Diff and merge in code](./images/helmfun12.png?raw=true "Diffing and merging for adding templating")


5. Now let's check our charts to make sure they're valid via the Helm lint function (you should still be in the roar-web directory).

```
helm lint --with-subcharts
```

What messages do you see?  What does the error mean?

6. There's a "nil pointer evaluation" because we haven't yet defined anything in values.yaml for "Values.image.repository" or "Values.image.tag".   Let's fix that now.

```   
code -d ../extra/values5-1.yaml charts/roar-db/values.yaml 
```
 
7.  Now we'll update our values.yaml file with the needed changes.  To save trying to get the yaml all correct in a regular editor, we’ll just use the diff tool’s merging ability. In the diff window, between the two files, click the arrow that points right to replace the code in our values.yaml file with the new code from extra/values5-1.yaml.  (In the figure below, this is the arrow that is circled and labelled "1".) After that, the files should be identical and you can close the diff window (circled "2" in the figure below).

![Diff and merge in code](./images/helmfun13.png?raw=true "Diffing and merging for adding templating")

 
8. After completing the merge, run the lint check again.  

```
helm lint --with-subcharts
```

This time you should not see any errors (though you will still see INFO messages).

9. Create a new namespace and deploy this new version of the chart.

```
k create ns roar2 
helm install roar2 -n roar2 .
```

10. Find the nodeport and open up the webapp from the release in a browser.

```
k get svc -n roar2
```

Find the NodePort value (will be > 30000)

11. Start the roar-port.sh command  with the namespace and nodeport.

```
../extra/roar-port.sh roar2 <nodeport>  &  
```

12. After executing this command, you'll see a popup in the lower right with a button to click on to see the application running. (If the dialog goes away, you can click on the *PORTS* tab in the top "tab" line of the terminal, find the row with the node port in the *Port* column, and click on that to open it up in a browser.)	

![Opening app via dialog](./images/helmfun9.png?raw=true "Opening app via dialog")
   

13. After this, you should get a browser tab with the web server running. To see the application running, add "/roar/" to the end of the URL.  **Make sure to include the trailing slash in /roar/**

![Opening app in browser](./images/helmfun10.png?raw=true "Opening app in browser")

14. Let's suppose we want to overwrite the image used here to be one that is for a test database. The image for the test database is on the quay.io hub at *quay.io/bclaster/roar-db-test:v4* . We could use a long command line string such as this to set it and use the template command to show the rendered files.  In the roar-web subdirectory, run the commands below to see the difference. 

```
helm template . --debug | grep image
helm template . --debug --set roar-db.image.repository=quay.io/bclaster/roar-db-test   --set roar-db.image.tag=v4  |  grep image
```

15. It's not always convenient to have to override things on the command line.  But there are other ways to override values.  Let's create a simple "extra" values file.   

```
code test-db.yaml
```

In the editor, add the following lines:

```
roar-db:
  image: 
    repository: quay.io/bclaster/roar-db-test
    tag: v4
```
**Close/save the file by clicking on the `x` in its tab.**

(For convenience, there is a test-db.yaml file in the "extra" area of the helm-fun-v2 area.)

16. Now, in one of your terminal windows, start a watch of the pods in your deployed helm release.  This is so that you can see the changes that will happen when we upgrade.  

```
k get pods -n roar2 --watch
```

17. Finally, let's do an upgrade using the new values file.  In a separate terminal window from the one where you did step 14, execute the following commands (this assumes you're still in the `roar-web` subdir):

```
helm upgrade -n roar2 roar2 . -f test-db.yaml --recreate-pods
```

Watch the changes happening to the pods in the terminal window with the watch running.

18. Run the roar-port.sh command to do the port forward again. (It may be a minute before the other one stops.)  Note that this is specifying the "roar2" namespace, not the "roar" one.

```
../extra/roar-port.sh roar2 <nodeport>   
```
 
19. Go back to your browser and refresh it.  You should see a version of the (TEST) data in use now. (Depending on how quickly you refresh, you may need to refresh more than once.)

![App with test data](./images/helmfun19.png?raw=true "App with test data")
 
20. Go ahead and stop the watch from running in the window via Ctrl-C.

```
Ctrl-C
```

<p align="center">
**[END OF LAB]**
</p>


**Lab 6: Using Functions and Pipelines**

**Purpose: In this lab, we'll see how to use functions and pipelines to expand what we can do in Helm charts.**

1. In our last lab, we added a separate values file that we could use to swap out which database we used - the prod one or the test one.   Now let's work on a way to do this based on a simple command line setting rather than having to have a separate file with multiple values.  To start, let's make a copy of our working project that already functions.  And then go to that directory.

```
cd /workspaces/helm-fun-v2
cp -R roar-web roar-web2
cd roar-web2
```

2. For switching between the prod and test database, we'll add two different versions of the image to choose from - one for prod and one for test.  Edit and modify the charts/roar-db/values.yaml file and add the following values. (For reference, or if you're concerned about the typing, in the ~/helm-fun-v2/extra folder, there is a values.yaml with the code already in it.) You can save and close the editor afterwards.

```   
code -d ../extra/values6-1.yaml charts/roar-db/values.yaml 
```

3. Now we'll update our values.yaml file with the needed changes.  To save trying to get the yaml all correct in a regular editor, we’ll just use the diff tool’s merging ability. In the diff window, between the two files, click the arrow that points right to replace the code in our values.yaml file with the new code from extra/values6-1.yaml.  (In the figure below, this is the arrow that is circled and labelled "1".) After that, the files should be identical and you can close the diff window (circled "2" in the figure below).

![Diff and merge in code](./images/helmfun14.png?raw=true "Diffing and merging for adding prod and test")

4. Now, we'll add some processing to enable selecting one of these images based on passing a setting for "stage" of either "PROD" or "TEST".  The design logic is 
```
if  stage = PROD
	use   image.prod.repository :  image.prod.tag
else if  stage = TEST
	use   image.test.repository : image.test.tag
else
   	display error message that we don't have a valid stage setting
```

Translating this into templating syntax, we use an if/(else if)/else flow.  The "eq" function is used for comparison, passing the two things to compare after it.  Also, there is a "required" function that will handle the default error checking.  Putting it all together, it could look like this:

```
{{- if eq .Values.stage  "PROD" }}
  image: "{{- .Values.image.prod.repository }}:{{ .Values.image.prod.tag -}}"
{{ else if eq .Values.stage  "TEST" }}
  image: "{{- .Values.image.test.repository }}:{{ .Values.image.test.tag -}}"
{{ else }}
  image: "{{- fail "A valid .Values.stage entry required!"  }}"
{{ end -}}
```

5. Next, update the deployment yaml file with this change. To save trying to get the yaml all correct in a regular editor, we’ll just use the diff tool’s merging ability. In the diff window, between the two files, click the arrow that points right to replace the code in our deployment.yaml file with the new code from extra/deployment6-1.yaml.  (In the figure below, this is the arrow that is circled and labelled "1".) After that, the files should be identical and you can close the diff window (circled "2" in the figure below). 

```
code -d  ../extra/deployment6-1.yaml charts/roar-db/templates/deployment.yaml 
```
![Diff and merge in code](./images/helmfun14.png?raw=true "Diffing and merging for adding prod and test")

6. Do a dry-run to make sure the image comes out as expected.   To simplify seeing the change, we'll grep for image with a few lines of context (from the roar-web2 subdir).

```
helm install --set roar-db.stage=TEST --dry-run foo . | grep image -n3
```
	You should see output like the following:
```
65-    spec:
66-      containers:
67-      - name: roar-db
68:        image: "quay.io/bclaster/roar-db-test:v4"
69:        imagePullPolicy: Always
70-        ports:
71-        - name: mysql
72-          containerPort: 3306
--
```

7. While this appears to work, what happens if we pass a lower-case value?

```
helm install --set roar-db.stage=test --dry-run foo . | grep image -n3
```

This is not what we want.   Let's make our arguments case-insensitive.  To do this, we'll pipe the value we pass in through a pipeline and to another template function called "upper" to upper-case it first.

8. Use the diff and merge functionality to merge in changes from ../extra/deployment6-2.yaml as done previously.

```
code -d ../extra/deployment6-2.yaml charts/roar-db/templates/deployment.yaml 
```

![Diff and merge in upper-case handling code](./images/helmfun16.png?raw=true "Diffing and merging for adding upper-case handling code")
 
9. Try running the command with the lower-case setting again. This time it should work.

```
helm install --set roar-db.stage=test --dry-run foo . | grep image -n3
```

10. What happens if we don't pass in a setting for stage?  Try it and see.

```
helm install --dry-run foo . | grep image -n3
```

We get an error because we don't have any value set and one of our functions fails because it doesn't have the correct type to compare.

11. We'd rather have a default that works.  Let's set one up in the values.yaml of the parent project that will be passed in to the child project. Use the diff and merge functionality to merge in changes from ../extra/values6-2.yaml as done previously.

```
code -d ../extra/values6-2.yaml values.yaml
```

![Diff and merge in defaults code](./images/helmfun17.png?raw=true "Diffing and merging for adding defaults")

  
12.	Now try running the same command as you did in step 8. 

```
helm install --dry-run foo . | grep image -n3
```

Notice that this time it worked.  Also notice that the value was passed down from the parent project.

13. Finally let's try deploying a running test instance of our application and a running instance of the prod version of our application. (

```
k create ns roar-prod
helm install --set roar-db.stage=PROD roar-prod -n roar-prod . 
k create ns roar-test
helm install --set roar-db.stage=TEST roar-test -n roar-test .
```

14. Now, you can get the NodePort for both versions, and open them in a browser to see the results.

```    
k get svc -n roar-prod | grep web
../extra/roar-port.sh roar-prod <nodeport for prod> (if not running in vm)
k get svc -n roar-test | grep web
../extra/roar-port.sh roar-test <nodeport for test> (if not running in vm)
```

15. As a reminder, the port numbers that are > 30000 are the ones you want.  You can open each of them in a browser as done before and view the different instance.

<p align="center">
**[END OF LAB]**
</p>

<p align="center">
(c) 2023 Brent Laster and Tech Skills Transformations
</p>
