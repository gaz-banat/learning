

https://itnext.io/understanding-kubernetes-maven-plugins-image-xml-configuration-e98ef633e231

-----------------------------------
kubernetes-maven-plugin - GOALS
-----------------------------------

First run mvn k8s:help to get all the goals of this plugin
Then run mvn k8s:help -Ddetail=true -Dgoal=<goal-name> to display parameter details.


Build Goals

	k8s:resource								-	generate the kubernetes manifests 						the lifecycle phase this plugin binds to is process-resources

		Configuration for this goal happens in 3 ways:

			1. Zero-Config


			2. XML plugin configuration

				<configuration>
					<resources>				
						<labels>
			 				...
						</labels>
						<annotations>
							...
						</annotations>
						<controller>			-	configuration for a controller, see below what a controller means
							...
						</confroller>
						<serviceAccount>		-	serviceAccount name which will be used by pods created by controller resources (e.g. Deployment, Cronjob, etc.)
							...
						</serviceAccount>
					</resources>			
				</configuration>


			3. Kubernetes & OpenShift resource fragments (external YAML files)
	
				The directory to use is src/main/jkube/
				src/main/jkube/<resource>.yml	
				e.g. src/main/jkube/cronjob.yml

				The name of the resource file will become the kind field in the file
				e.g. if you name the file src/main/jkube/job.yaml then its kind will be job

		Output
			resources are produced in target/classes/META-INF/jkube/ directory
			there should be a kubernetes.yaml manifest with all the resources in it
			there should be a kubernetes/ directory with the individual manifests

	
	k8s:build									-	build an image with the source code

		EITHER 	you need access to a docker engine
					EITHER 	use a current unix socket 
					OR		the following environment variables
									export DOCKER_TLS_VERIFY="1"
									export DOCKER_HOST="tcp://192.168.64.5:2376"
									export DOCKER_CERT_PATH="/Users/gaz/.minikube/certs"
									export MINIKUBE_ACTIVE_DOCKERD="minikube"

		OR		use jib mode to do a build
					-Djkube.build.strategy=jib					-	set the property at runtime
					 <properties>
						<jkube.build.strategy>jib</jkube.build.strategy>	-	set the propert in the pom.xml
					</properties>


		OR		use Dockerfile mode - a Dockerfile in the project root directory next to pom. kubernetes-maven-plugin will detect it and do a build
					optionally -Djkube.image.name=“<image-name>” 


		Provide the name of the image to build at runtime:
		-Djkube.generator.name="ghazanfar/code-with-quarkus:0.3"
	

	k8s:push								-	push the image to a registry



	k8s:apply								-	apply the manifests to the currently logged in cluster					the lifecycle phase this plugin binds to is install


Development Goals

	k8s:deploy				-		deploy resource descriptors to a cluster after creating them and building the app. Same as k8s:apply except that it runs in the background

	k8s:undeploy				-		undeploy and remove resources from a cluster

	k8s:log					-		show the logs of the running application

	k8s:debug					-		enable remote debugging

	k8s:watch					-		watch for file changes and perform rebuilds and deployments


Other Goals

	k8s:helm

	k8s:helm-push

	k8s:migrate

	k8s:help

	k8s:remote-dev


-----------------------------------
kubernetes-maven-plugin - CONFIGURATION
-----------------------------------




------------------------
Terms
------------------------

In JKube terminology a ‘controller’ resource is a kubernetes resource which manages pods. Can be one of the following
ReplicationController
ReplicaSet
Deployment
StatefulSet
DaemonSet
Job
CronJob
