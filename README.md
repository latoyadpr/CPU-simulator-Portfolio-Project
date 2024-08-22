# CPU-simulator-Portfolio-Project

class ALU:
    def add(self, a, b):
        return a + b
    
    def sub(self, a, b):
        return a - b
    
    # Additional ALU operations...

class Cache:
    def __init__(self, size):
        self.cache = [None] * size
    
    def load(self, address):
        # Cache read logic
        return self.cache[address % len(self.cache)]
    
    def store(self, address, value):
        # Cache write logic
        self.cache[address % len(self.cache)] = value

class MemoryBus:
    def __init__(self, memory_size, cache_size):
        self.memory = Memory(memory_size)
        self.cache = Cache(cache_size)
    
    def load(self, address):
        value = self.cache.load(address)
        if value is None:
            value = self.memory.load(address)
            self.cache.store(address, value)
        return value
    
    def store(self, address, value):
        self.memory.store(address, value)
        self.cache.store(address, value)

class Memory:
    def __init__(self, size):
        self.memory = [0] * size
    
    def load(self, address):
        return self.memory[address]
    
    def store(self, address, value):
        self.memory[address] = value

class CPU:
    def __init__(self, memory_bus):
        self.alu = ALU()
        self.memory_bus = memory_bus
        self.pc = 0
        self.registers = [0] * 8
        self.ir = 0
    
    def fetch(self):
        self.ir = self.memory_bus.load(self.pc)
        print(f"Fetched instruction 0x{self.ir:X} from address {self.pc}")
        self.pc += 1
    
    def decode_execute(self):
        instruction = self.ir >> 26
        rs = (self.ir >> 21) & 0x1F
        rt = (self.ir >> 16) & 0x1F
        rd = (self.ir >> 11) & 0x1F
        immediate = self.ir & 0xFFFF
        
        if instruction == 0x00:  # R-type instructions
            funct = self.ir & 0x3F
            if funct == 0x20:  # ADD
                self.registers[rd] = self.alu.add(self.registers[rs], self.registers[rt])
                print(f"Executed ADD: R{rd} = R{rs} + R{rt}")
            elif funct == 0x22:  # SUB
                self.registers[rd] = self.alu.sub(self.registers[rs], self.registers[rt])
                print(f"Executed SUB: R{rd} = R{rs} - R{rt}")
            # Additional R-type instructions...
        
        elif instruction == 0x23:  # LW
            self.registers[rt] = self.memory_bus.load(self.registers[rs] + immediate)
            print(f"Executed LW: R{rt} = Memory[R{rs} + {immediate}]")
        
        elif instruction == 0x2B:  # SW
            self.memory_bus.store(self.registers[rs] + immediate, self.registers[rt])
            print(f"Executed SW: Memory[R{rs} + {immediate}] = R{rt}")
        
        # Additional I-type and J-type instructions...
    
    def run(self):
        while True:
            self.fetch()
            self.decode_execute()

def load_instructions(cpu, filename):
    with open(filename, 'r') as file:
        address = 0
        for line in file:
            instruction = int(line.strip(), 16)
            cpu.memory_bus.memory.store(address, instruction)
            address += 1

def initialize_memory_bus(memory_bus, filename):
    with open(filename, 'r') as file:
        for line in file:
            address, value = map(int, line.strip().split())
            memory_bus.store(address, value)

# Example program loading and execution:
memory_bus = MemoryBus(memory_size=256, cache_size=16)
cpu = CPU(memory_bus)

# Load instructions and initial memory values
load_instructions(cpu, 'instructions.txt')
initialize_memory_bus(memory_bus, 'memory_init.txt')

cpu.run()
