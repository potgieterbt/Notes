[Source](https://github.com/RPCSX/rpcsx/tree/master/rpcsx-os)

main.cpp:
* main function:
	* [[rpcsx-os#^2dc507| rx::vfs::initialize]] - I think this just inits the memory.
	* Set some variables.
	* Loop though the arguments:
		* First argument mounts firmware to a directory using [[rpcsx-os#^44c921 | rx::vfs::mount]].
		* Checks if tracing is enabled for this particular run.
		* Checks if running as root for this particular run.
		* Checks if isSystem is set.
		* Checks if override is set -> [[rpcsx-os#^b79c1d | rx::linker::override]]
		* Checks if audio should be enabled.
	* Initialize thread -> [[rpcsx-os#^a46390| rx::thread::initialize]]
	* Initialize VM -> [[rpcsx-os#^64a2ab | rx::vm::initialize]]
	* Run rpcsxgpu -> [[rpcsx-os#^736eb6 | runRpsxGpu]]
	* If audio enabled setup audio in orbis kernel
	* [[orbis-kernel#^9299ff | Allocate]] a Pid and [[orbis-kernel#^71bb1a | create]] a process for Pid <- if isRoot is set Pid = 1 else Pid = 10. Change name to 10.MAINTHREAD.
	* Start a thread that does:
		* Change thread name to Bridge
		* Sets local bridge var to [[amdgpu#^b8e19f | rx::bridge]].header
		* Creates vector of uint64_t and reserves memory of size of the variable cacheCommands.
		* Creates while true loop:
			* for command in cacheCommands:
				* Loads value from command using relaxed memory ordering (I think because it is in a thread).
				* if value is not 0 load into fetchedCommands.
				* Stores 0 into command using relaxed memory ordering.
			* Check if fetchedCommands is empty if it is then restart previous loop.
			* For each command in fetchedCommands:
				* Sets page to the uint32_t cast of the command (perhaps the lower half of a 64 bit value).
				* Sets count to count bit shifted 32 bits to the right (perhaps the upper half of a 64 bit value).
				* Sets pageFlags to the value loaded from the cachedPages vector at index page using relaxed memory ordering.
				* Sets address to the uint64_t cast of page multiplied by the [[amdgpu#^aad6ea | amd::bridge::kHostPageSize]]
				* Sets origVmProt (original VM protection?) to [[rpcsx-os#^e2ef34 | rx::vm::getPageProtection]](address)



rx::vfs::initialize ^2dc507

rx::vfs::mount ^44c921

rx::linker::override ^b79c1d

rx::thread::initialize ^a46390

rx::vm::initialize ^64a2ab

runRpsxGpu ^736eb6

rx::vm::getPageProtection ^e2ef34