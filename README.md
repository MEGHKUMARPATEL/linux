
Name : Dhruv Khut
SJSU ID : 018228041

Name : Meghkumar Patel 
SJSU ID :  018219461


Course Name : CMPE-283 Sec 48 - Virtual Technologies
Assignment Number : 2
1. Team Contributions
Dhruv Khut
I was responsible for setting up the GCP environment, including configuring the "outer" and "inner" VMs. I forked and cloned the Linux kernel repository, built the kernel, and installed it on the outer VM. I also monitored the logs using dmesg to analyze exit patterns and contributed to documenting the steps and observations in this assignment.
Meghkumar Patel
Megh worked on locating and modifying the KVM exit handler code to add the necessary counters and logging functionality. He implemented the logic to print exit statistics every 10,000 exits, rebuilt the kernel with the changes, and ensured the inner VM ran smoothly with the modified KVM code.
2. Steps to Complete the Assignment
Step 1: Kernel Setup on GCP
Create GCP VMs


Set up two GCP instances: an "outer" VM with nested virtualization enabled and an "inner" VM to run inside it.
Verified that both VMs were running smoothly with appropriate network configurations.
Fork and Clone the Linux Repository


Forked the Linux Kernel GitHub Repository into my GitHub account.
Cloned the repository into the outer VM:
git clone https://github.com/MEGHKUMARPATEL/linux 
cd linux  


Build and Install the Kernel


Built the kernel and installed modules:

make -j$(nproc)  
sudo make modules_install  
sudo make install  
Reboot with the New Kernel


Rebooted the outer VM to use the new kernel:
sudo reboot  
uname -r  
Step 2: Modifying the KVM Code
Locate and Modify the Exit Handler Code
Opened the KVM source file:
 cd arch/x86/kvm  
nano vmx.c  
Added counters and logic to log exit statistics every 10,000 exits. 

Example:
static unsigned long long kvm_exit_counters[256] = {0}; 
static unsigned long long kvm_total_exits = 0;       
static const char *kvm_exit_reason_to_string(int reason) {
    switch (reason) {
    case 0: return "EXCEPTION_NMI";
    case 1: return "EXTERNAL_INTERRUPT";
    case 2: return "TRIPLE_FAULT";
    case 9: return "CPUID";
    case 28: return "HLT";
    case 30: return "MSR_WRITE";
    case 48: return "EPT_MISCONFIG";
    default: return "UNKNOWN_EXIT";
    }
}


static int __vmx_handle_exit(struct kvm_vcpu *vcpu, fastpath_t exit_fastpath) {
    struct vcpu_vmx *vmx = to_vmx(vcpu);
    union vmx_exit_reason exit_reason = vmx->exit_reason;
    u32 vectoring_info = vmx->idt_vectoring_info;
    u16 exit_handler_index;

  int exit_type = exit_reason.basic; // Get the exit type number
    kvm_exit_counters[exit_type]++;
    kvm_total_exits++;

    if (kvm_total_exits % 10000 == 0) {
        int i;
        for (i = 0; i < 256; i++) {
            if (kvm_exit_counters[i] > 0) {
                printk(KERN_INFO "KVM Exit: %d (%s) occurred %llu times\n",
                       i, kvm_exit_reason_to_string(i), kvm_exit_counters[i]);
            }
        }
    }

Rebuild and Install the Modified Kernel


Rebuilt the kernel with changes:

 make -j$(nproc)  
sudo make modules_install  
sudo make install  
sudo reboot  

Step 3: Testing the Modified Kernel
Boot the Inner VM


Started the inner VM using the modified kernel in the outer VM.
Monitor Logs


Used dmesg to check for exit statistics:
 dmesg | grep "KVM Exit Statistics"  
3. Questions
Q1: Team Contributions
See the Team Contributions section above.
Q2: Steps Description
The steps are detailed in the Steps to Complete the Assignment section. Each step has been written in a way that anyone can follow to reproduce the assignment on GCP.
Q3: Frequency of Exits
The frequency of exits generally increases steadily. However, spikes occur during specific operations like I/O or when the inner VM is booting. From our observations, a full VM boot involved approximately 85,000 exits.
Q4: Most and Least Frequent Exit Types
Most Frequent Exit Types: Timer-related exits and I/O operations. These were consistently logged the most during testing.
Least Frequent Exit Types: Exits like HLT_EXIT appeared very rarely in the logs.



![App Screenshot](https://github.com/MEGHKUMARPATEL/linux/blob/master/screenshot/screenshot%201.jpg)


![App Screenshot](https://github.com/MEGHKUMARPATEL/linux/blob/master/screenshot/screenshot%202.jpg)

