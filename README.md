## Create-a-datacenter-with-one-host-and-run-one-cloudlet-on-it.


## Experiment 


# Create a Datacenter with One Host and Run One Cloudlet (CloudSim)

This repository demonstrates a **basic CloudSim simulation** where a single datacenter with one host is created, and one cloudlet is executed on it. The project is intended for **cloud computing laboratory experiments** and beginner-level understanding of CloudSim architecture.

---

## Aim

To create a simple cloud computing environment using CloudSim by configuring one datacenter with a single host and executing one cloudlet, and to observe the simulation results.



## Objectives

* Understand the working of the CloudSim toolkit
* Learn how to create a datacenter and host in CloudSim
* Execute a single cloudlet on a virtual machine
* Analyze cloudlet execution output

---

##  Software Requirements

* Operating System: Windows / Linux / macOS
* Java Development Kit (JDK): 8 or above
* IDE: Eclipse / IntelliJ IDEA / NetBeans
* CloudSim Toolkit: Version 3.x

---

##  Hardware Requirements

* Processor: Intel i3 or higher
* RAM: Minimum 4 GB
* Storage: At least 10 GB free disk space

---

## Project Structure

```
Create-a-datacenter-with-one-host-and-run-one-cloudlet-on-it/
│
├── src/
│   └── DatacenterSingleHostSingleCloudlet.java
├── lib/
│   └── cloudsim-3.x.jar
├── output/
│   └── simulation_output.png
├── README.md
```

---



##  Steps to Run the Project

1. Install Java JDK (8 or above).
2. Download and configure the CloudSim library.
3. Import the project into your IDE.
4. Add CloudSim JAR files to the project build path.
5. Compile and run the Java program.
6. Observe the cloudlet execution results in the console.

---

##  Simulation Workflow

1. Initialize CloudSim
2. Create a Datacenter with one Host
3. Create a Broker
4. Create one Virtual Machine (VM)
5. Create one Cloudlet
6. Bind the Cloudlet to the VM
7. Start Simulation
8. Display Results

---
## Program


package org.cloudbus.cloudsim.examples;


import org.cloudbus.cloudsim.*;
import org.cloudbus.cloudsim.core.CloudSim;
import org.cloudbus.cloudsim.provisioners.BwProvisionerSimple;
import org.cloudbus.cloudsim.provisioners.PeProvisionerSimple;
import org.cloudbus.cloudsim.provisioners.RamProvisionerSimple;

import java.text.DecimalFormat;
import java.util.ArrayList;
import java.util.Calendar;
import java.util.LinkedList;
import java.util.List;

public class Exp1_RA183 {
    public static DatacenterBroker broker;

    private static List<Cloudlet> cloudletList;
    private static List<Vm> vmlist;

    public static void main(String[] args) {
        Log.println("Starting CloudSimExample1...");

        try {
            // First step: Initialize the CloudSim package. It should be called before creating any entities.
            int num_user = 1; // number of cloud users
            Calendar calendar = Calendar.getInstance(); // Calendar whose fields have been initialized with the current date and time.
            boolean trace_flag = false; // trace events

            CloudSim.init(num_user, calendar, trace_flag);

            Datacenter datacenter0 = createDatacenter("Datacenter_0");

            // Third step: Create Broker
            broker = new DatacenterBroker("Broker");
            int brokerId = broker.getId();

            // Fourth step: Create one virtual machine
            vmlist = new ArrayList<>();

            int vmid = 0;
            int mips = 1000;
            long size = 10000; // image size (MB)
            int ram = 512; // vm memory (MB)
            long bw = 1000;
            int pesNumber = 1; // number of cpus
            String vmm = "Xen"; // VMM name

            Vm vm = new Vm(vmid, brokerId, mips, pesNumber, ram, bw, size, vmm, new CloudletSchedulerTimeShared());

            vmlist.add(vm);

            // submit vm list to the broker
            broker.submitGuestList(vmlist);

            // Fifth step: Create one Cloudlet
            cloudletList = new ArrayList<>();

            // Cloudlet properties
            int id = 0;
            long length = 400000;
            long fileSize = 300;
            long outputSize = 300;
            UtilizationModel utilizationModel = new UtilizationModelFull();

            Cloudlet cloudlet = new Cloudlet(id, length, pesNumber, fileSize,
                    outputSize, utilizationModel, utilizationModel,
                    utilizationModel);
            cloudlet.setUserId(brokerId);
            cloudlet.setGuestId(vmid);

            // add the cloudlet to the list
            cloudletList.add(cloudlet);

            // submit cloudlet list to the broker
            broker.submitCloudletList(cloudletList);

            // Sixth step: Starts the simulation
            CloudSim.startSimulation();

            CloudSim.stopSimulation();

            //Final step: Print results when simulation is over
            List<Cloudlet> newList = broker.getCloudletReceivedList();
            printCloudletList(newList);

            Log.println("CloudSimExample1 finished!");
        } catch (Exception e) {
            e.printStackTrace();
            Log.println("Unwanted errors happen");
        }
    }

    private static Datacenter createDatacenter(String name) {

        List<Host> hostList = new ArrayList<>();

        List<Pe> peList = new ArrayList<>();

        int mips = 1000;

        peList.add(new Pe(new PeProvisionerSimple(mips)));
        int ram = 2048; // host memory (MB)
        long storage = 1000000; // host storage
        int bw = 10000;

        hostList.add(
                new Host(
                        new RamProvisionerSimple(ram),
                        new BwProvisionerSimple(bw),
                        storage,
                        peList,
                        new VmSchedulerTimeShared(peList)
                )
        );
        String arch = "x86";
        String os = "Linux";
        String vmm = "Xen";
        double time_zone = 10.0;
        double cost = 3.0;
        double costPerMem = 0.05;
        double costPerStorage = 0.001;
        double costPerBw = 0.0;
        LinkedList<Storage> storageList = new LinkedList<>();

        DatacenterCharacteristics characteristics = new DatacenterCharacteristics(
                arch, os, vmm, hostList, time_zone, cost, costPerMem,
                costPerStorage, costPerBw);

        Datacenter datacenter = null;
        try {
            datacenter = new Datacenter(name, characteristics, new VmAllocationPolicySimple(hostList), storageList, 0);
        } catch (Exception e) {
            e.printStackTrace();
        }

        return datacenter;
    }

    private static void printCloudletList(List<Cloudlet> list) {
        int size = list.size();
        Cloudlet cloudlet;

        String indent = "    ";
        Log.println();
        Log.println("========== OUTPUT ==========");
        Log.println("Cloudlet ID" + indent + "STATUS" + indent
                + "Data center ID" + indent + "VM ID" + indent + "Time" + indent
                + "Start Time" + indent + "Finish Time");

        DecimalFormat dft = new DecimalFormat("###.##");
        for (Cloudlet value : list) {
            cloudlet = value;
            Log.print(indent + cloudlet.getCloudletId() + indent + indent);

            if (cloudlet.getStatus() == Cloudlet.CloudletStatus.SUCCESS) {
                Log.print("SUCCESS");

                Log.println(indent + indent + cloudlet.getResourceId()
                        + indent + indent + indent + cloudlet.getGuestId()
                        + indent + indent
                        + dft.format(cloudlet.getActualCPUTime()) + indent
                        + indent + dft.format(cloudlet.getExecStartTime())
                        + indent + indent
                        + dft.format(cloudlet.getExecFinishTime()));
            }
        }
    }
}







##  Output

The simulation displays:

* Cloudlet ID
* Execution status
* Datacenter ID
* VM ID
* Start time
* Finish time

 

##  Output Screenshot

<img width="1059" height="978" alt="image" src="https://github.com/user-attachments/assets/9b1c9c75-2524-435d-bbe1-8d71257fdb60" />


---

##  Applications

* Cloud Computing Laboratory Experiments
* Learning CloudSim Basics
* Academic Demonstrations

---

##  Conclusion

This project successfully demonstrates the creation of a simple cloud environment using CloudSim, where a single cloudlet is executed on one host. It provides a strong foundation for understanding more complex cloud simulations.

---

 

---

##  License

This project is intended for educational and academic use only.
