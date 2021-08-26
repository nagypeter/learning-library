# Explore Verrazzano and Other Consoles

## Introduction

In Lab 3, we deployed the Bob's Book application. In this lab, we will access the application and verify that it is working. Later, we will explore the Verrazzano and Grafana consoles.

### Objectives

In this lab, you will:

* Access the Bob's Book application.
* Explore the Verrazzano console.
* Explore the Grafana console.

### Prerequisites

* Run Lab 1, which creates an OKE Cluster on the Oracle Cloud Infrastructure.
* Run Lab 2, which installs Verrazzano on a Kubernetes cluster.
* Run Lab 3, which deploys the Bob's Book application.
* You should have a text editor, where you can paste the commands and URLs and modify them, as per your environment. Then you can copy and paste the modified commands for running them in the *Cloud Shell*.

## Task 1: Access the Bob's Book application

1. We need an `EXTERNAL_IP` address through which we can access the Bob's Book application. To get the `EXTERNAL_IP` address of the istio-ingressgateway service, copy the following command and paste it in the *Cloud Shell*.

      ```bash
      <copy> kubectl get service \
      -n "istio-system" "istio-ingressgateway" \
      -o jsonpath={.status.loadBalancer.ingress[0].ip}; echo</copy>
      ```

   ![hostName](images/1.png)

2. To open the Robert's Book Store Home Page, copy the following URL and replace *XX.XX.XX.XX* with your *EXTERNAL_IP* address which we got in the last step, as shown in the following image.

      ```bash
      <copy>https://bobs-books.bobs-books.XX.XX.XX.XX.nip.io/</copy>
      ```

   ![Roberts Books Page](images/2.png)

3. Click *Advanced*, as shown:

   ![Advanced](images/3.png)

4. Select *Proceed to bobs-books.bobs-books. EXTERNAL_IP .nip.io(unsafe)* to access the application.

   ![Unsafe access](images/4.png)

   ![Robert Bookstore](images/5.png)

5. To open the Bob's Book Store Home page, open a new tab and copy the following URL and replace *XX.XX.XX.XX* with your `EXTERNAL_IP` address, as shown in the following image.

      ```bash
      <copy>https://bobs-books.bobs-books.XX.XX.XX.XX.nip.io/bobbys-front-end/</copy>
      ```

   ![Bobs bookstore](images/6.png)

   ![Bob bookstore](images/7.png)

   > Leave this page open  because we will use it in Lab 8.

6. To open the Bob's Book Order Manager UI, open a new tab and copy the following URL and replace *XX.XX.XX.XX* with your *EXTERNAL_IP* address as shown in the following image.

      ```bash
      <copy>https://bobs-books.bobs-books.XX.XX.XX.XX.nip.io/bobs-bookstore-order-manager/orders</copy>
      ```

   ![order manager](images/8.png)

7. Go Back to the *Bob's Books* page and let's purchase a book. Click *Books* as shown in the following image.

   ![Check out order](images/38.png)

8. Select the image for the *Twilight* Book, as shown in the following image.

   ![Purchase book](images/39.png)

9. First, click *Add to cart* and then *Checkout* as shown in the following image.

   ![Purchase book](images/40.png)

10. Enter the details for purchasing the book. For *Your State*, enter your two digit state code and then click *Submit Order*.

   ![Purchase book](images/42.png)
11. Go back to the *Order Manager* page and select the *Refresh* button to check if your order is successfully recorded in the order manager.

   ![Verify Order](images/11.png)

## Task 2: Explore the Verrazzano Console

Verrazzano installs several consoles. The endpoints for an installation are stored in the `Status` field of the installed Verrazzano Custom Resource.

1. You can get the endpoints for these consoles by using the following command and looking at the `Status.Instance` field:

      ```bash
      <copy>kubectl get vz -o yaml</copy>
      ```

   ![Console URL](images/12.png)

   This results in output similar to the following (output abbreviated to show only the relevant portions). Click the links to open the *Verrazzano* Console as shown:

   ![Verrazzano Console](images/13.png)

2. Click *Advanced*.

   ![Advanced](images/14.png)

3. Select *Proceed to verrazzano default XX.XX.XX.XX.nip.io(unsafe)*.

   ![Proceed](images/15.png)

4. Because it redirects to the Keycloak console  URL for authentication, again on this page, click *Advanced*.

   ![Keycloak Authentication](images/16.png)

5. Select *Proceed to Keycloak default XX.XX.XX.XX.nip.io(unsafe)*.

   ![Proceed](images/17.png)

6. Now we need the user name and password for the Verrazzano console. *Username* is *verrazzano* and to find out the password, go back to the *Cloud Shell* and paste the following command to find out the password for the *Verrazzano Console*.

      ```bash
      <copy>kubectl get secret --namespace verrazzano-system verrazzano -o jsonpath={.data.password} | base64 --decode; echo</copy>
      ```

   ![Verrazzano Password](images/18.png)

7. Copy the password and go back to the browser, where the *Verrazzano Console* is open.

   ![Verrazzano login](images/36.png)

8. Paste the password in the *Password* field and enter *verrazzano* as *Username* and then click *Sign In*.

   ![SignIn](images/19.png)

   In the Home Page of the Verrazzano Console, you can see *System Telemetry* and because we installed the *Development Profile* of Verrazzano, you can see it in the *General Information* section.

   ![Home Page](images/20.png)

9. Because we have deployed the Bob's Book application, you can see it under *OAM Applications*. Select *bobs-books* to view different components of this application.

   ![Components](images/21.png)

   There are 10 components for this application as you can see under *Components*.

10. To explore the configuration for a particular component, first select the *Sort by* drop-down menu and select *Name*, then select the *bobby-coh* component as shown:

    ![bobs-coh](images/22.png)

11. You can see *General Information* for this component. To learn about the *Workload Spec*, select *bobby-coh* as shown:

    ![Workload spec](images/23.png)

12. Here you can see the configuration details for the *bobby-coh* component. Click *Close*.

    ![configuration](images/24.png)

## Task 3: Explore the Grafana Console

1. Select *Home* to go back to Verrazzano Console Home Page.

    ![Home](images/25.png)

2. In this page, you'll see the link for opening the *Grafana console*. Select the link for the *Grafana Console* as shown:

    ![Grafana Home](images/26.png)

3. Click *Advanced*.

    ![Advanced](images/27.png)

4. Select *grafana.vmi.system.default.XX.XX.XX.XX.nip.io(unsafe)*.

    ![proceed](images/28.png)

5. The Grafana Home Page opens. Select *Home*  at the top left.

    ![Home](images/29.png)

6. Type *WebLogic* and you will see *WebLogic Server Dashboard* under *General*. Select *WebLogic Server Dashboard*.

    ![WebLogic](images/30.png)

    Here you can observe the two domains under *Domain* and Running Servers, Deployed Applications, Server Name and their Status, Heap Usage, Running Time, JVM Heap. If your application has resources like JDBC and JMS, you can also get details about it here.

    ![Dashboard](images/31.png)

7. Now, select WebLogic Server Dashboard and type *Helidon* and you will see *Helidon Monitoring Dashboard*. Select *Helidon Monitoring Dashboard*.

    ![Helidon](images/32.png)

    Here you can see various details like the *Status* of your application and its *Uptime*, Garbage Collector, and Mark Sweep Total and its Time, Thread Count.

    ![Helidon Dashboard](images/33.png)

8. Now, select Helidon Monitoring Dashboard and type *Coherence* and you will see *Coherence Dashboard Main*. Select *Coherence Dashboard Main*.

    ![Coherence](images/34.png)

9. Here you can see the details of the *Coherence Cluster*. For the Bob's Books application, we have two Coherence clusters, one for Bob's Books and another for Robert's Books. You need to select the drop-down menu for *Cluster Name* to view both the clusters.

    ![Dashboard](images/35.png)
    ![Dashboard](images/37.png)



Leave the *Cloud Shell* open; we will use it for upcoming labs.

## Acknowledgements

* **Author** -  Ankit Pandey
* **Contributors** - Maciej Gruszka, Peter Nagy
* **Last Updated By/Date** - Kamryn Vinson, July 2021
