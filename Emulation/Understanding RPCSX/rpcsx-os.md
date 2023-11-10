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



rx::vfs::initialize ^2dc507

rx::vfs::mount ^44c921

rx::linker::override ^b79c1d

rx::thread::initialize ^a46390

rx::vm::initialize ^64a2ab

runRpsxGpu ^736eb6