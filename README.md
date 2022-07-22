# UCP_In_Campsim

How to Run : 

    Firstly we have to make a "run_2core.sh" file which will help us run champsim for 2 core.
    
    Now there will be a folder named dpc3_traces where we have to put the traces we want to run for out simulation.
    
    Then,we have to open the terminal in that directory itself.
    
    Now, first build the champsim using the following command : "./build_champsim.sh bimodal no no no no lru 2"
    
    After building the champsim file, now run the 2core file using the following command :
    
    "./run_2core.sh bimodal-no-no-no-no-lru-2core 1 10 0 gcc_13B.trace.xz gobmk_135B.trace.xz"

    NOTE :
    
    The last two terms in the command are the traces that we are running in this dual core system.
    
           We can change them accordingly for the respective two cores.
           
           We can change the number of simulations also by replacing the "10" in the command by our own number.
           

    The output file will be created in the folder named "results_2core_10M".
    
    A file named "mix0-bimodal-no-no-no-no-lru-2core.txt" will be generated in this folder which contains our test results.

Implementation of UCP : 


    What is UCP?
    
    Utility-Based Cache Partitioning is a low-overhead runtime approach that divides a shared cache between several programmes based on the expected         reduction in cache misses for a given quantity of cache resources.
    
    The suggested technique uses a unique, cost-effective hardware circuit that requires less than 2kB of storage to monitor each application at runtime. 
    A partitioning algorithm uses the data obtained by the monitoring circuits to determine the quantity of cache resources given to each application.
    The goal of this project is to integrate UCP-based cache partitioning into Champsim, a trace-based simulator.

    How we implemented UCP?
    We have created following modules under :

    1) Replacement Policy for LLC
        i) lru_victim_llc :  This function is basically to find which block we have to evict for the incoming new block. 
                             This policy will act according to partition we have created in the LLC.
                             Firstly if there is any Invalid block or empty block in the LLC, it will directly evict that block for the insertion of new block. 
                             Then, we have created two counter in this function which will count the number of block of each core present in a given set of LLC.
                             Then on the basis of the incoming block of the particular trace and the values of the counters and our current partition we will
                             decide which block has the maximum LRU age.
                             This block will be the one which has to be evicted.

        ii) lru_update_llc : This function updates the lru age of the blocks present in a particular set on the basis of requesting core.
                             If the following block is the latest block inserted then it will promote it to the MRU position by assigning its LRU age to 0,
                             else it will just increase the LRU age of the blocks by 1. 
                             
    2. Auxillary Tag Directory (ATD): 
    (i) Created a class ATD in Cache.cc : 
            class ATD {
                public:
                    uint64_t tag;
                    uint32_t lru;

                    ATD() {
                        tag = 0;
                        lru = 0;
                    };
                }; 
            This class has ATD tag and lru as members and a constructor ATD setting lru and tag bit to zero.
    (ii) atd_llc: void atd_llc(uint32_t set, const PACKET *packet, uint32_t cpu);
            This function checks word accessed is hit in ATD or not, if it is then increments the hit counter for that LRU position
            and put the incoming block at MRU position. It also increments the age of blocks by one whose age are less 
            than the incoming block age.
    (iii) miss_llc: void miss_llc(const PACKET *packet, uint32_t set, uint32_t cpu);
            This function checks wheteher the incoming block is present in ATD or not, then putting it at MRU position and 
            increments the age of all other blocks.

    3. After every 5 million cycles UMON is giving the partition based on utility and replacement policy of LLC is behaving according 
    this.
