Download Link: https://assignmentchef.com/product/solved-solvedproject-2-accessing-eeprom-via-i2c-bus-solution
<br>
Exercise Objectives1. To learn the basic programming technique in Linux i2c and gpio kernel modules.2. To learn i2c and gpio driver architecture and software3. To apply an application framework to control read/write operations of EEPROM.Lab AssignmentMost operating systems provide memory mapping facility that allows users to directly access memory device by associating a range of user-space addresses to device memory. When a user program reads from or writes to the assigned address range, the OS performs the function by accessing either a buffered copy in memory or the memory device. This is so called the mmap device or the mmap abstraction which is built upon a device driver to read or write blocks from a memory device (such as a memory card or a frame buffer).In this assignment, you will develop a driver to perform page read and write operations for an EEPROM. The memory device is a Microchip 24FC256 EEPROM which consists of 512 64-byte pages (or blocks) which can be accessed via an I2C bus. To connect an EEPROM chip, we will use SCL and SDA pins of the digital port on Galileo board. The signals are from the I2C_gpio controller of Quark processor. They are multiplexed and selected with the I2C_MUX signal of the Cypress IO expander on Galileo board. Thus, to enable the I2C bus access to the EEPROM, it is necessary to set up the correct gpio signal on the IO expander. The following diagram shows the connections to an external Microchip 24FC256 EEPROM. Note that the i2c address of the chip is set to 0x54 by connecting its A2 pin to high.The 1st task of the assignment is to build an i2c client driver for the EEPROM device (i2c_flash) such that user programs can invoke read, write, and ioctl driver operations. You can assume the EEPROM chip has a fixed I2C address and is connected to a specific i2c bus. Thus, when the module is initialized, the EEPROM device should be created and named as “i2c_flash”. The module should enable device file operations to support the following user-level calls:1. int open(const char *pathname, int flags);2. ssize_t read(int fd, void *buf, size_t count); /*to read a sequence of count pages from the EEPROM device into the user memory pointed by buf. The pages to be read start from the current page position of the EEPROM. The page position is then advanced by count and, if reaching the end of pages, wrapped around to the beginning of the EEPROM. */ 3. ssize_t write(int fd, const void *buf, size_t count); /* to write a sequence of count pages to an EEPROM device starting from the current page position of the EEPROM. The page position is then advanced by count and, if reaching the end of pages, wrapped around to the beginning of the EEPROM. */IO ExpanderGalileo boardEEPROMSCL SDAQuarkSCLSDAI2C_MUXanalogswitchGIP4. int ioctl(int fd, unsigned long request, …); /* if request=FLASHGETS, the call returns the status of the EEPROM (busy or not). If a call is with request=FLASHGETP or FLASHSETP, the call gets or sets the current page position with a third argument int *argp. Proper return code should be set for the call, including busy or out of memory. The 3rd request, when request=FLASHERASE and the EEPROM is not busy, is to trigger an erase operation to the EEPROM which write all 1’s to all 512 pages. */5. int close(int fd);where count and offset are page number of the EEPROM memory and is ranged from 0 to 511, and the size of the “buf” should be 64*count bytes. Also, the read and write calls are blocking calls, i.e., they will be returned only when the operations are done (return 0) or have an error (return -1), e.g., the EEPROM is busy. Hence, the calling user thread is blocked while the requested operation is in progress.The 2nd task of the lab assignment is to revise your i2c_flash device driver of task 1 such that the read and write calls are non-blocking. To provide the asynchronous processing of EEPROM functions, a work queue will be created when the driver module i2c-flash is installed. When a read system call is requested to the EEPROM device, the driver takes one of the following operations: If no EEPROM data is ready for read and the EEPROM is not busy, the driver submits a read transfer request to the work queue and returns immediately with -1 and an error code ERRNO=EAGAIN. If no EEPROM data is ready for read and the EEPROM is busy, the call returns immediately with -1 and an error code ERRNO=EBUSY. If EEPROM data is ready for read, return with 0 after the data of 64*count bytes is copied to the buffer.On the other hand, when a write system call is requested, the driver: Returns immediately with 0 if the EEPROM is not busy and the write operation will be performed by the driver subsequently. If the EEPROM is busy, returns immediately with -1 and an error code ERROR=EBUSY.Note that, instead of using the work queue mechanism, you may choose to use kernel threads to handle non-blocking calls.In addition to the operations on the EEPROM, your driver should light up a led whenever a read, write, or erase operation is in progress, i.e. the EEPROM is busy. The led should be connected to Galileo’s digital IO pin IO8. To test your driver, you will need to develop a user application that can exercise all driver functions.