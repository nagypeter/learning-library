# Verify the Changes in the Bob's Book Application, Verrazzano Console, and Grafana Console

## Introduction

In Lab 7, we applied changes in bobbys-helidon-stock-application and the pod for it is in the *Running* state. In this lab, we will verify the changes in the application, the Verrazzano Console, and the Grafana Console.

### Objectives

In this lab, you will:

* Verify the changes in the Bob's Books application.
* Verify the changes in the Verrazzano Console.
* Verify the changes in the Grafana Console.

### Prerequisites

* You should have a text editor, where you can paste the commands and URLs and modify them, as per your environment. Then you can copy and paste the modified commands for running them in the *Cloud Shell*.

## Task 1: Verify the Changes in Bob's Books Application

1. Open the Bob's Books tab and select Refresh. If you have closed that tab, then copy and paste the following command in the text editor and replace the XX.XX.XX.XX  with the EXTERNAL_IP for the application. You will notice that the Book Name is in all upper case letters.

    ```bash
    <copy>https://bobs-books.bobs-books.XX.XX.XX.XX.nip.io/bobbys-front-end/</copy>
    ```

    ![Bob's Book](images/1.png)

## Task 2: Verify the Changes in the Verrazzano Console

1. You determined the endpoints for the Verrazzano Console as part of Lab 4, but, if you want to get the link for the Verrazzano console, copy the following command and run it in the *Cloud Shell*.

    ```bash
    <copy>kubectl get vz -o yaml</copy>
    ```

    You can see the link for the Verrazzano Console, select it to open the console.

    ![Verrazzano Console](images/2.png)

2. In Lab4, if you saved the password for the Verrazzano Console, you can use it to log in to the Verrazzano Console. Otherwise, run the following command in the *Cloud Shell* to get the password and paste it in your text editor.

    ```bash
    <copy>kubectl get secret --namespace verrazzano-system verrazzano -o jsonpath={.data.password} | base64 --decode; echo</copy>
    ```

    ![Verrazzano Password](images/3.png)

3. Enter *verrazzano* as the user name and use the output of the previous command from the text editor, as the password then click *Sign In*.

    ![Sign In](images/4.png)

4. Select the application name, 'bobs-books', under OAM Applications as shown below:

    ![bobs-book application](images/5.png)

5. Select the drop-down menu to sort by Name and then select the component Name, *bobby-helidon*.

    ![Bobby Helidon](images/6.png)

6. In the previous lab, we modified the *bobs-books-comp.yaml* file which provides the specification for every component. Now, in the Verrazzano console, we will verify the changes that we made in Lab 7. Select *bobby-helidon* in *Workload Spec*.

    ![Workload spec](images/7.png)

    You will see that this component is using the new Docker image from the Oracle Cloud Container Registery repository.

    ![New Repository](images/8.png)

## Task 3: Verify the Changes in the Grafana Console

1. Select *Home* to go back to the Verrazzano Home Page.

    ![Home](images/9.png)

2. Select the link for Grafana to open the *Grafana Console*.

    ![Grafana](images/10.png)

3. Select Home, as shown, type *Helidon*, and then select *Helidon Monitoring Dashboard*.

    ![Helidon](images/11.png)

    ![Helidon Dashboard](images/12.png)

4. In the ServiceID, select *bobs-books_default_bobs-books_bobby-helidon* and in the instance, select the newly created instance. In this case, you will get information for the modified *bobby-helidon-stock-application*.

    ![New Component](images/13.png)

Congratulations!

You have successfully completed the labs.

## Acknowledgements

* **Author** -  Ankit Pandey
* **Contributors** - Maciej Gruszka, Peter Nagy
* **Last Updated By/Date** - Kamryn Vinson, July 2021