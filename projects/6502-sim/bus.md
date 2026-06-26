
## Header

Creating a class Bus. Includes Constructor, Destructor, CPU Object and RAM. RAM is just an array of uint8_t of 64 kB.

Read and Write methods are present. Write writes uint8_t data to uint16_t address (64 kB). Read return uint8_t data based on uint16_t address argument.

## Source

### Constructor
Calls CPU connect bus interface and initializes RAM to 0x00.

### Destructor
default

### Write
Checks bounds and stores data in RAM array at addr.

### Read
Checks bounds and returns data from array RAM at addr.