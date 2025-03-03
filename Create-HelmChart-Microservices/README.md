# Create HelmChart for Microservices 

**What is Helm Chart**

```
  - Helm Chart is a bundle of a bunch of Yaml file .

  - When I deploy an App with those Configuration yaml file I can bundle it for the next use . I don't have to do Configuration again
```

**3 Ways to create Helm Chart***

```
  1. Create separate Helm Chart when Configuration is very different

  2. Create 1 Helm Chart when Configurations is very the same

  3. Combination of both Options
```

**Basic Structure of HelmChart**

- Create HelmChart `helm create <helm-chart-name>`

- Inside Helm Chart Folder :

  - **chart.yaml** : Describe metadata
 
  - **Chart Folder**: Chart Folder hold any dependencies charts that these charts might have
 
  - **.helmignore**:
    - File don't want to include in my Helm Chart
    - Basically this file tell Helm to ignore all of these Folder and Files when building a Package Archive from this chart . Helm Chart can build and packaged and distributed just like any other artifacts
   
  - **Template Folder**:
    - This is a core Helm Chart
   
    - This is where actual K8s Yaml file created
   
    - .yaml file : The attribute same as in the K8s Service configuration file . Instead of hardcode value I have placeholder defined as {{}} the value of this syntax are actually the placeholder to actual Value 









  
