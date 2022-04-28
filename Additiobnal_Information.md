# Additional Information

In order to get started, there are some things that are important to understand. The first is some useful scripts for adding and removing resources. The second is a brief explanation of the conf files that will be used.

## Scripts

There are 7 scripts that will be commonly used. Add_zephyr_res, add_zephyr_route, get_zephyr_resources, get_zephyr_routes, get_zephyr_topo, post_zephy_res and remove_zephyr_res. All scripts use curl in order to achieve their task.

### add_zephyr_res
This script [add_zephyr_res](https://github.com/linux-genz/genz-utils/blob/master/add_zephyr_res) is used to add an additional consumer to an existing resource. The script requires 3 arguments. The first argument must be a resource conf file. The second argument must be the host url. The third argument must be the instance UUID originally returned by post_zephyr_res.

### add_zephyr_route
This script [add_zephyr_route](https://github.com/linux-genz/genz-utils/blob/master/add_zephyr_route) is used to add a route. The first argument must be a route conf file. The second argument must be the host url.

### get_zephyr_resources
This script [get_zephyr_resources](https://github.com/linux-genz/genz-utils/blob/master/get_zephyr_resources) is used to obtain the resource file and save it to a desired file. The first argument must be the file where the resource file will be saved. The second argument must be the host url. 

### get_zephyr_routes
This script [get_zephyr_routes](https://github.com/linux-genz/genz-utils/blob/master/get_zephyr_routes) is used to obtain the routes file and save it to a desired file. The first argument must be the file where the routes file will be saved. The second argument must be the host url.

### get_zephyr_topo
This script [get_zephyr_topo](https://github.com/linux-genz/genz-utils/blob/master/get_zephyr_topo) is used to obtain the fabric topology file and save it to a desired file. The first argument must be the file where the fabric topology file will be saved. The second argument must be the host url.

### post_zephyr_res
This script [post_zephyr_res](https://github.com/linux-genz/genz-utils/blob/master/post_zephyr_res) is used to add a resource to a host. The script requires 2 arguments. The first argument must be a resource conf file. The second argument must be the host url.

### remove_zephyr_res
This script [remove_zephyr_res](https://github.com/linux-genz/genz-utils/blob/master/remove_zephyr_res) is used to remove a resource from a host. The script requires 3 arguments. The first argument must be a resource conf file. The second argument must be the host url. The third argument must be the instance UUID originally returned by post_zephyr_res.

## Conf files

### Example Zephyr-fabric-conf

An example conf file that includes comments about each field is located in the genz-utils directory. It is found either at /path/to/genz_utils/zephyr-fm/example-zephyr-fabric.conf or  [example-zephy-fabric.conf](https://github.com/linux-genz/genz-utils/blob/master/zephyr-fm/example-zephyr-fabric.conf) though it is important to note that the comments make this example not usable. In order to use this file the comments need to be removed and the instructions to fill out each field need to be followed. Additionally this file may be useful for future reference after a zephyr-fabric.conf file is already set up.

### Individual component conf

The fields are similar to the fabric conf, but for a single resource. This is a file that needs to be provided to the scripts listed below for adding and removing components.

### Conf fields
see example for layout and where the below fields are placed
fabric_uuid - This is the fabric uui. Can be generated via a command line with `python -c \"import uuid; print(uuid.uuid4())\"`

#### producer
This field is CUUID:SerialNum for a target ZMM. This value can be obtained via `lsgenz`.

#### consumers
This field is the CUUID:SerialNum for your bridge. If the resource is going to be shared, add all CUUID:SerialNum values in the list.

#### class_uuid
This field is used to indicate either genz-blk or dax-genz class_uuid
for genz-blk use 3cb8d3bd-51ba-4586-835f-3548789dd906
for dex-genz use f147276b-c2c1-431e-91af-3031d0039768

#### flags
flags that indicate class specific information
genz-blk this field must be 0
dax-genz bits 23:16 target numa mode
         bits 15:08 dax region id
         bits 07:00 dax id

#### instance_uuid
This field is generated at runtime by zephyr and is service specific.

#### class
This field is the Gen-Z component class. See Appendix C of the GenZ Core spec for more details. Examples:
2 is Memory (Explicit OpClass)
17 is Block storage(Non-bootable)

#### start
The start of the memory resource

#### length
The capacity of the memory resource(currently limited to 126GiB visible to any 1 Orthus host)

#### type
Indicate the type of memory
type 0 is control space
type 1 is data space
type 2 is interleaved data space(planned)

#### ro_rkey
Region key to use in read-based request packets. Set to 0 as it is currently not supported.

#### rw_rkey
Region key to use in write based and interrupt request packets. Set to 0 as it is currently not supported.




[linux-genz]: https://github.com/linux-genz/linux  
[udk/orthus]: https://github.com/linux-genz/linux/udk/orthus  
[README]: https://github.com/linux-genz/udk/blob/master/README.md 
[Release_Notes]: https://github.com/linux-genz/linux/udk/orthus/Known_Bugs_and_Limitations.md  
[Known_Bugs_and_Limitations]: https://github.com/linux-genz/linux/udk/Known_Bugs_and_Limitations.md  

