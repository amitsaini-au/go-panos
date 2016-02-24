## go-panos
[![GoDoc](https://godoc.org/github.com/scottdware/go-panos?status.svg)](https://godoc.org/github.com/scottdware/go-panos) [![Travis-CI](https://travis-ci.org/scottdware/go-panos.svg?branch=master)](https://travis-ci.org/scottdware/go-panos)
[![license](http://img.shields.io/badge/license-MIT-red.svg?style=flat)](https://raw.githubusercontent.com/scottdware/go-panos/master/LICENSE)

A Go package that interacts with Palo Alto and Panorama devices using the XML API.

Be sure to visit the [GoDoc][godoc-go-panos] page for official package documentation.

> Note: The below examples are a work-in-progress :)

### Examples

* [Connecting to a device][connecting-to-a-device]
* [Listing objects (address, service, device-groups, tags, templates, etc.)][listing-objects]
* [Creating objects][creating-objects]
* [Deleting objects][deleting-objects]
* [Applying/removing tags from objects][tagging-objects]
* [Modifying groups (address, service, etc.)][modifying-groups]
* [Renaming objects][renaming-objects]
* [Commiting configurations][commiting-configurations]

#### Connecting to a Device

Establish a connection to a Palo Alto firewall or Panorama is pretty straightforward:

```Go
pa, err := panos.NewSession("pa200-fw", "admin", "paloalto")
if err != nil {
    fmt.Println(err)
}
```

Once you are connected, some basic information about the firewall/session is established. The fields returned are as follows:

|Field|Description|
|-----|-----------|
|Host|Hostname/IP of the device.|
|Key|Encrypted key for API access.|
|URI|The base URI that all API calls will use.|
|Platform|Hardware platform of the device.|
|Model|Model number of the device.|
|Serial|Serial number of the device.|
|SoftwareVersion|Software version currently active on the device.|
|DeviceType|Type of device, i.e. "panorama" or "panos."|
|Panorama|A boolean (true/false) field that determines if the device is/isn't Panorama.| 

```Go
fmt.Printf("Host: %s\n", pa.Host)
fmt.Printf("Key: %s\n", pa.Key)
fmt.Printf("URI: %s\n", pa.URI)
fmt.Printf("Platform: %s\n", pa.Platform)
fmt.Printf("Model: %s\n", pa.Model)
fmt.Printf("Serial: %s\n", pa.Serial)
fmt.Printf("Software Version: %s\n", pa.SoftwareVersion)
fmt.Printf("Device Type: %s\n", pa.DeviceType)
fmt.Printf("Panorama Connection: %t\n", pa.Panorama)
```

#### Listing Objects

##### Devices (Panorama)

> Note: Panorama ONLY

To list all devices managed by a Panorama device, use the `Devices()` function. It will return a list of 
device serial numbers that you can iterate over.

|Field|Description|
|-----|-----------|
|Serial|Serial number of the device.|

```Go
devices, _ := pa.Devices()

for _, d := range devices.Devices {
    fmt.Println(d.Serial)
}
```

##### Address Objects and Groups

To list all address objects, use the `Addresses()` function. The fields returned are as follows: 

> Note: Some of these fields will be empty based on the type of address returned.

|Field|Description|
|-----|-----------|
|Name|Name of the object.|
|IPAddress|IP/network of the object.|
|IPRange|IP range of the object.|
|FQDN|Fully qualified domain name of the object.|
|Description|Description of the object, if available.|

```Go
addrs, _ := pa.Addresses()

for _, a := range addrs.Addresses {
    fmt.Println(a.Name)
    fmt.Println(a.IPAddress)
    fmt.Println(a.IPRange)
    fmt.Println(a.FQDN)
    fmt.Println(a.Description)
}
```

To list all address group objects, use the `AddressGroups()` function. The fields returned are as follows:

|Field|Description|
|-----|-----------|
|Name|Name of the group.|
|Type|Type of address group, i.e. "Static" or "Dynamic."|
|DynamicFilter|The filter for the dynamic address group.|
|Members|Address objects that belong to the group.|
|Description|Description of the group, if available.|

> Note: Members are in a string slice (`[]string`), so to iterate over them you can just do another loop.

```Go
addrGroups, _ := pa.AddressGroups()

for _, ag := range addrGroups.Groups {
    fmt.Println(ag.Name, ag.Type, ag.DynamicFilter, ag.Description)
    for _, m := range ag.Members {
        fmt.Println(m)
    }
}
```

##### Service Objects and Groups

To list all service objects, use the `Services()` function. The fields returned are as follows:

> Note: Some fields might be empty based on they type of service returned.

|Field|Description|
|-----|-----------|
|Name|Name of the object.|
|TCPPort|TCP port(s) of the object.|
|UDPPort|UDP port(s) of the object.|
|Description|Description of the object, if available.|

```Go
svcs, _ := pa.Services()

for _, s := range svcs.Services {
    fmt.Println(s.Name)
    fmt.Println(s.TCPPort)
    fmt.Println(s.UDPPort)
    fmt.Println(s.Description)
}
```

To list all service groups, use the `ServiceGroups()` function. The fields returned are as follows:

|Field|Description|
|-----|-----------|
|Name|Name of the object.|
|Members|Service objects that belong to the group.|
|Description|Description of the object, if available.|

> Note: Members are in a string slice (`[]string`), so to iterate over them you can just do another loop.

```Go
svcGroups, _ := pa.ServiceGroups()

for _, sg := range svcGroups.Groups {
    fmt.Println(sg.Name, sg.Description)
    for _, m := range sg.Members {
        fmt.Println(m)
    }
}
```

##### Device Groups

> Note: Panorama ONLY

To list all device-groups on a Panorama device, use the `DeviceGroups()` function. The fields returned are as follows:

|Field|Description|
|-----|-----------|
|Name|Name of the object.|
|Devices|Individual devices that belong to the device-group.|

> Note: Devices (serial numbers) are in a string slice (`[]string`), so to iterate over them you can just do another loop.

```Go
devGroups, _ := pa.DeviceGroups()

for _, d := range devGroups.Groups {
    fmt.Println(d.Name)
    for _, serial := range d.Devices {
        fmt.Println(serial)
    }
}

```

##### Templates

> Note: Panorama ONLY

To list all templates on a Panorama device, use the `Templates()` function. The fields returned are as follows:

|Field|Description|
|-----|-----------|
|Name|Name of the object.|
|Devices|Individual devices that the template is applied to.|

> Note: Devices (serial numbers) are in a string slice (`[]string`), so to iterate over them you can just do another loop.

```Go
temps, _ := pa.Templates()

for _, t := range temps.Templates {
    fmt.Println(t.Name)
    for _, serial := range t.Devices {
        fmt.Println(serial)
    }
}

```

##### Tags

To list all tags, use the `Tags()` function. The fields returned are as follows:

|Field|Description|
|-----|-----------|
|Name|Name of the tag.|
|Color|Color of the tag, i.e. "Blue" or "Cyan."|
|Comments|Description of the tag, if available.|

```Go
tags, _ := pa.Tags()

for _, t := range tags.Tags{
    fmt.Println(t.Name)
    fmt.Println(t.Color)
    fmt.Println(t.Comments)
}
```

#### Creating Objects

##### Adding Devices (Panorama)

> Note: Panorama ONLY

To add a device to be managed by Panorama, you can use the `AddDevice()` function. This takes the following parameters:

* `serial` - Serial number of the device
* `devicegroup` (optional) - If specified, the device will be added to Panorama, as well as the given device-group.

> Note: If you specify a device-group, and the device does not already exist within Panorama...then it will be added to Panorama as well.

```Go
// Add a device to Panorama
pa.AddDevice("1084782033")

// Add a device to a device-group
pa.AddDevice("1084782033", "Lab-Device-Group")
```

##### Addresses

The `CreateAddress()` function takes 4 parameters: `name`, `address type`, `address` and an (optional) `description`. When
creating an address object on Panorama, you must specify the `device-group` to create the object in as the last parameter.


> Note: The second parameter is the address type. It can be one of `ip`, `range` or `fqdn`.

```Go
pa.CreateAddress("fqdn-object", "fqdn", "sdubs.org", "My personal website")
pa.CreateAddress("Apple-subnet", "ip", "17.0.0.0/8", "")
pa.CreateAddress("IP-range", "range", "192.168.1.1-192.168.1.20", "IP address range")
```

When creating an address or service on a Panorama device, specify the desired device-group as the 
last parameter, like so:

```Go
pa.CreateAddress("panorama-IP-object", "ip", "10.1.1.5", "", "Lab-Devices")
```

##### Address Groups

The `CreateStaticGroup()` function creates a static address group, and takes the following parameters: `name`, `members`, `description`. When
creating an address group on Panorama, you must specify the `device-group` to create the object in as the last parameter.

If you are specifying multiple address objects, they must be separated by a comma for the `members` parameter: `"server1, server2, pc1"`

```Go
pa.CreateStaticGroup("Custom-address-objects", "my-ip1, server-subnet, fqdn-host", "")

// When creating an address group on a Panorama device, specify the desired device-group as the 
// last parameter, like so:
pa.CreateStaticGroup("Custom-address-objects", "my-ip1, server-subnet, fqdn-host", "", "Lab-Device-Group")
```

The `CreateDynamicGroup()` function creates a dynamic address group, and takes the following parameters: `name`, `criteria`, `description`. When
creating an address group on Panorama, you must specify the `device-group` to create the object in as the last parameter.

The `criteria` parameter must be written similar to how it is once you have selected your match criteria (tags) through the GUI:

`'tag1' and 'tag2' or 'tag5'`

> Note: You can get a listing of tags on the device by using the `Tags()` function as described above.

```Go
pa.CreateDynamicGroup("Dynamic-Servers", "'server-tag' and 'other tag'", "")

// Creating a dynamic address group on a Panorama devices is done like so:
pa.CreateDynamicGroup("Dynamic-Servers", "'server-tag' and 'other tag'", "", "Some-Panorama-Device-Group")
```

##### Services

The `CreateService()` function takes 4 parameters: `name`, `protocol`, `port` and an (optional) `description`. When
creating a service object on Panorama, you must specify the `device-group` to create the object in as the last parameter.

> Note: The third parameter is the port number(s). Port can be a single port #, range (1-65535), or comma separated (80, 8080, 443).

```Go
pa.CreateService("proxy-ports", "tcp", "8080,80,443", "")
pa.CreateService("misc-udp", "udp", "10001-10050", "Random UDP ports")
```

Just like creating address objects on Panorama, the same applies to service objects:

```Go
pa.CreateAddress("panorama-ports", "tcp", "8000-9000", "Misc TCP ports", "Lab-Devices")
```

##### Service Groups

The `CreateServiceGroup()` function creates a service group, and takes the following parameters: `name`, `members`, `description`. When
creating a service group on Panorama, you must specify the `device-group` to create the object in as the last parameter.

If you are specifying multiple service objects, they must be separated by a comma for the `members` parameter: `"tcp-port, udp-port"`

```Go
pa.CreateServiceGroup("Custom-ports", "tcp-5000, web-browsing, snmp", "")

// When creating a service group on a Panorama device, specify the desired device-group as the 
// last parameter, like so:
pa.CreateServiceGroup("Custom-ports", "tcp-5000, web-browsing, snmp", "", "Lab-Device-Group")
```

##### Tags

The `CreateTag()` function creates a tag on the device, and takes the following parameters: `name`, `color`, `comments`. When
creating a tag on Panorama, you must specify the `device-group` to create the object in as the last parameter.

`color` can be any one of the following: Red, Green, Blue, Yellow, Copper, Orange, Purple, Gray, Light Green, Cyan, Light Gray, 
Blue Gray, Lime, Black, Gold, Brown.

```Go
pa.CreateTag("vm-servers", "Red", "VMware servers")

// When creating a tag on a Panorama device, specify the desired device-group as the 
// last parameter, like so:
pa.CreateTag("vm-servers", "Red", "VMware servers", "Lab-Device-Group")
```

##### Device Groups

> Note: Panorama ONLY

To create a new device-group on a Panorama device, you can use the `CreateDeviceGroup()` function. This takes two parameters: `name` and `description`.
If you don't want a description, then you can just use empty double-quotes `""`. An optional third parameter is a `[]string` list of device serial
numbers that you want to add to the device group. To get a list of devices, you can always use the `Devices()` function.

If you do not want to add any devices to the device-group initially, then you can use `nil` as the last parameter value.

```Go
pa.CreateDeviceGroup("Remote-Office-Firewalls", "All remote office locations", nil)

// Add devices to the new group from the start. Create a []string list:
devices := []string{
    "1093822222",
    "1084782033",
    "1084783947",
    "1654783433",
}

pa.CreateDeviceGroup("Campus Firewalls", "", devices)
```

#### Deleting Objects

Deleting objects is just as easy as creating them. Just specify the object name, and if deleting objects from Panorama,
specify the device-group as the last parameter.

```Go
// Delete address objects
pa.DeleteAddress("fqdn-object")
pa.DeleteAddress("some-panorama-IP", "Lab-Device-Group")

// Delete a service object
pa.DeleteService("proxy-ports")

// Delete address group objects
pa.DeleteAddressGroup("Addr-Group-Name")
pa.DeleteAddressGroup("Panorama-address-group", "Lab-Device-Group")

// Delete a service group
pa.DeleteServiceGroup("web-browsing-ports")

// Delete tags from the device
pa.DeleteTag("server-tag")
pa.DeleteTag("server-tag", "Lab-Device-Group")

// Delete a device-group from Panorama
pa.DeleteDeviceGroup("Lab-Devices")

// Remove a device from a specific device-group
pa.RemoveDevice("1084782033", "Some-Device-Group")

// Remove a device from Panorama
pa.RemoveDevice("1084782033")
```

#### Tagging Objects

You can apply tags to address and service objects by using the `ApplyTag()` function. It takes two parameters: `tag` and `object`. 
When tagging an object on a Panorama device, you must specify the `device-group` to create the object in as the last parameter.

`tag` can have multiple values, and they must be separated by a comma, i.e. `"server-tag, lab, warehouse"`. If you have multiple objects with
the same name, then all of them that match will have the tag(s) applied.

```Go
pa.ApplyTag("web", "fqdn-object")

// Use multiple tags on an object
pa.ApplyTag("internet, web, proxy", "proxy-ports")

// Tag a Panorama object
pa.ApplyTag("servers, virtual", "server-farm", "Production-Device-Group")
```

##### Removing Tags

To remove a tag from an object, use the `RemoveTag()` function. This function takes two parameters: `object` and `tag`. You can only remove a
single tag at a time. When removing a tag from an object on a Panorama device, you must specify the `device-group` to create the object in as the last parameter.

```Go
pa.RemoveTag("web", "fqdn-object")

// Remove tag from a Panorama object
pa.RemoveTag("servers", "server-farm", "Production-Device-Group")
```

#### Modifying Groups

To modify an address or service group, which includes adding/removing members...you can use the `ModifyGroup()` function, which takes 4 parameters: `objecttype`, 
`action`, `object` and `group`. When modifying a group object on a Panorama device, you must specify the `device-group` to create the object in as the last parameter.

* `objecttype` is one of: "address" or "service"
* `action` is one of: "add" or "remove"

`object` and `group` are the names of the objects, respectively.

```Go
pa.ModifyGroup("address", "add", "ip-range", "Corporate-Subnets")
pa.ModifyGroup("service", "remove", "proxy-ports", "Web-Browsing")

// Modify a group on a Panorama device
pa.ModifyGroup("address", "add", "my-laptop", "Security-Folks", "Panorama-Device-Group")
```

#### Renaming Objects

To rename any address, service, or tag object...use the `RenameObject()` function. You only have to specify the following parameters: `oldname` and `newname`.
When renaming an object on a Panorama device, you must specify the `device-group` to create the object in as the last parameter.

```Go
pa.RenameObject("my-ip", "my-new-ip")
pa.RenameObject("server-tag", "web-server-tag")

// To rename an object on a Panorama device, specify the device-group as the last parameter
pa.RenameObject("proxy-ports", "legacy-proxy-ports", "Panorama-Device-Group")
```

#### Commiting Configurations

There are two commit functions: `Commit()` and `CommitAll()`. Commit issues a normal commit on the device. When issuing `Commit()` against a Panorama device,
the configuration will only be committed to Panorama, and not an individual device-group.

Using `CommitAll()` will only work on a Panorama device, and you have to specify the device-group you want to commit the configuration to. You can
also selectively commit to certain devices within that device group by adding their serial numbers as additional parameters.

```Go
pa.Commit()

// CommitAll will commit the configuration to the given device-group, and all of it's devices.
pa.CommitAll("Lab-Device-Group")

// Commit to only 2 devices in a device group - you MUST use their serial numbers
pa.CommitAll("Lab-Device-Group", "1093822222", "1084782033")
```

[godoc-go-panos]: http://godoc.org/github.com/scottdware/go-panos
[license]: https://github.com/scottdware/go-panos/blob/master/LICENSE
[connecting-to-a-device]: https://github.com/scottdware/go-panos#connecting-to-a-device
[listing-objects]: https://github.com/scottdware/go-panos#listing-objects
[creating-objects]: https://github.com/scottdware/go-panos#creating-objects
[deleting-objects]: https://github.com/scottdware/go-panos#deleting-objects
[commiting-configurations]: https://github.com/scottdware/go-panos#commiting-configurations
[tagging-objects]: https://github.com/scottdware/go-panos#tagging-objects
[modifying-groups]: https://github.com/scottdware/go-panos#modifying-groups
[renaming-objects]: https://github.com/scottdware/go-panos#renaming-objects