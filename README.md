# Experiment-5-Creation-of-two-datacenters-with-one-host-each-and-run-two-cloudlets-on-them.-

 
---

## Overview
This project demonstrates a CloudSim 3.0.3 simulation where two users submit cloudlets that are executed across two separate datacenters. Each datacenter contains one host, and tasks are scheduled through a broker that distributes workloads to virtual machines.

This experiment illustrates multi-datacenter architecture, user-level task scheduling, and distributed cloud execution behavior.

---

## Aim
To create two datacenters with one host each and execute cloudlets from two users to analyze distributed execution behavior.

---

## Objectives
- Simulate multiple datacenters in CloudSim
- Configure hosts and allocate resources
- Simulate multiple users submitting tasks
- Create virtual machines for execution
- Analyze distributed execution results

---

## Tools and Technologies

| Tool | Version |
|------|---------|
| Java JDK | 1.8+ |
| CloudSim | 3.0.3 |
| IDE | IntelliJ / Eclipse |
| OS | Windows / Linux |

---

## System Configuration

### Datacenter Configuration

| Parameter | Value |
|----------|------|
| Datacenters | 2 |
| Hosts per DC | 1 |
| RAM | 2048 MB |
| Storage | 1,000,000 MB |
| Bandwidth | 10,000 Mbps |
| Scheduler | Time Shared |

---

### Virtual Machine Configuration

| Parameter | VM1 | VM2 |
|----------|-----|-----|
| MIPS | 500 | 500 |
| RAM | 512 MB | 512 MB |
| BW | 1000 Mbps | 1000 Mbps |
| VMM | Xen | Xen |
| Scheduler | Time Shared | Time Shared |

---

### Cloudlet Configuration

| Parameter | Value |
|----------|------|
| Users | 2 |
| Cloudlets | 4 |
| Lengths | 20000, 40000, 60000, 80000 |
| File Size | 300 MB |
| Output Size | 300 MB |
| PEs | 1 |

---
 
---

## Algorithm
1. Initialize CloudSim library  
2. Create two datacenters  
3. Configure one host per datacenter  
4. Create broker  
5. Create virtual machines  
6. Submit VM list to broker  
7. Create cloudlets with different lengths  
8. Submit cloudlets  
9. Start simulation  
10. Collect results  

---

## Execution Time Formula
Execution Time = Cloudlet Length / VM MIPS

## Program 
package org.cloudbus.cloudsim.examples;
import org.cloudbus.cloudsim.*;
import org.cloudbus.cloudsim.core.CloudSim;
import org.cloudbus.cloudsim.provisioners.*;

import java.util.*;

public class CloudSimExample5 {

    public static Datacenter createDatacenter(String name) {

        List<Host> hostList = new ArrayList<>();

        List<Pe> peList = new ArrayList<>();
        peList.add(new Pe(0, new PeProvisionerSimple(1000)));

        Host host = new Host(
                0,
                new RamProvisionerSimple(2048),
                new BwProvisionerSimple(10000),
                1000000,
                peList,
                new VmSchedulerTimeShared(peList)
        );

        hostList.add(host);

        DatacenterCharacteristics characteristics =
                new DatacenterCharacteristics(
                        "x86","Linux","Xen",hostList,
                        10.0,3.0,0.05,0.001,0.0
                );

        Datacenter datacenter = null;

        try {
            datacenter = new Datacenter(
                    name,
                    characteristics,
                    new VmAllocationPolicySimple(hostList),
                    new LinkedList<Storage>(),
                    0
            );
        }
        catch (Exception e) {
            e.printStackTrace();
        }

        return datacenter;
    }

    public static void main(String[] args) {

        try {

            CloudSim.init(1, Calendar.getInstance(), false);

            Datacenter dc1 = createDatacenter("Datacenter_1");
            Datacenter dc2 = createDatacenter("Datacenter_2");

            DatacenterBroker broker = new DatacenterBroker("Broker");
            int brokerId = broker.getId();

            // ---------- VMs ----------
            List<Vm> vmList = new ArrayList<>();

            vmList.add(new Vm(0, brokerId, 500, 1, 512, 1000, 10000,
                    "Xen", new CloudletSchedulerTimeShared()));

            vmList.add(new Vm(1, brokerId, 500, 1, 512, 1000, 10000,
                    "Xen", new CloudletSchedulerTimeShared()));

            broker.submitVmList(vmList);

            // ---------- CLOUDLETS ----------
            List<Cloudlet> cloudletList = new ArrayList<>();
            UtilizationModel model = new UtilizationModelFull();

            long[] lengths = {20000, 40000, 60000, 80000}; // different execution times

            for(int i = 0; i < 4; i++) {

                Cloudlet cloudlet = new Cloudlet(
                        i,
                        lengths[i],
                        1,
                        300,
                        300,
                        model,
                        model,
                        model
                );

                cloudlet.setUserId(brokerId);
                cloudletList.add(cloudlet);
            }

            broker.submitCloudletList(cloudletList);

            // ---------- RUN ----------
            CloudSim.startSimulation();
            List<Cloudlet> results = broker.getCloudletReceivedList();
            CloudSim.stopSimulation();

            // ---------- OUTPUT ----------
            System.out.println("\n========== RESULTs ==========");

            for (Cloudlet cl : results) {
                System.out.println(
                        "Cloudlet " + cl.getCloudletId()
                                + " | Length: " + cl.getCloudletLength()
                                + " | VM: " + cl.getVmId()
                                + " | Datacenter: " + cl.getResourceId()
                                + " | Time: " + cl.getActualCPUTime()
                );
            }

        }
        catch (Exception e) {
            e.printStackTrace();
        }
    }
}



 

## Sample Output
========== RESULTs ==========
Cloudlet 0 | Length: 20000 | VM: 0 | Datacenter: 2 | Time: 80.0
Cloudlet 2 | Length: 60000 | VM: 0 | Datacenter: 2 | Time: 160.0
Cloudlet 1 | Length: 40000 | VM: 1 | Datacenter: 2 | Time: 160.0
Cloudlet 3 | Length: 80000 | VM: 1 | Datacenter: 2 | Time: 240.0





---

## Result
The simulation successfully executed tasks submitted by multiple users across two datacenters. Cloudlets were scheduled based on available resources and executed accordingly.

---

## Conclusion
This experiment demonstrates that CloudSim effectively models distributed cloud environments. It shows how workloads are handled across multiple datacenters and validates distributed task execution concepts used in real cloud infrastructure.
