from m5.objects import *

# Define a system
system = System()

# Clock and voltage domain settings
system.clk_domain = SrcClockDomain()
system.clk_domain.clock = '1GHz'
system.clk_domain.voltage_domain = VoltageDomain()

# Memory system configuration
system.mem_mode = 'timing'  # Use timing mode for detailed simulations
system.mem_ranges = [AddrRange('512MB')]  # 512 MB of memory

# Define a simple CPU model with distinct pipeline stages
system.cpu = TimingSimpleCPU()

# Define a memory controller (DDR3)
system.membus = SystemXBar()  # Memory bus
system.mem_ctrl = DDR3_1600_8x8()
system.mem_ctrl.range = system.mem_ranges[0]
system.mem_ctrl.port = system.membus.master

# Connect CPU to memory
system.cpu.icache_port = system.membus.slave  # Instruction cache
system.cpu.dcache_port = system.membus.slave  # Data cache

# Add a workload
system.workload = SEWorkload.init_compatible("tests/test-progs/hello/bin/x86/linux/hello")  # A simple "Hello, World!" program

# Set up process for the workload
process = Process()
process.cmd = ["tests/test-progs/hello/bin/x86/linux/hello"]  # Path to the binary
system.cpu.workload = process
system.cpu.createThreads()

# Connect the system
system.system_port = system.membus.slave

# Instantiate and start the simulation
root = Root(full_system=False, system=system)  # Full-system mode is off
m5.instantiate()

print("Starting simulation...")
exit_event = m5.simulate()
print(f"Simulation ended at tick {m5.curTick()} because {exit_event.getCause()}")
