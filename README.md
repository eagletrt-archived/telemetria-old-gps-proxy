# telemetria-gps-proxy

The purpose of this repository is to allow broadcast communication from the GPS serial port to different applications simultaneously.

When the GPS is connected it's seen as `/dev/ttyACM0`, and only allows for one connection at a time.

**ttybus** it's a tool from [danielinux](https://github.com/danielinux/) that allows sharing of tty devices between different processes, and therefore satisfies our needs. 
The explanation of all of its functions is left to the [repository's documentation](https://github.com/danielinux/ttybus), this readme will show how it's implemented and used in our context.

##Create the fake bus at car boot up

Once the car and it's software components boot up, `fakebus.service` has to be launched to create a fake bus, after that `attachbus.service` can run to attach the bus to the GPS port `/dev/ttyACM0`.

This has to be done only once, the bus will be created in `/tmp` and will expire only when the `tty_bus` process is killed.

In the same way the bus will remain attached to the bus as long as the `tty_attach` process is running.

These two services are placed in `/etc/systemd/system` to automatically start at start up.

The bus is now broadcasting the signal coming from `/dev/ttyACM0` and it's ready to accept connections from fake devices created from now on.

##Create a fake device

Every time a new application needs to read GPS data, a new fake device need to be created and attached to the existing bus, to do that, use the `tty_fake` C program in the **ttybus** repository, indicating the bus to attach to, and the position of the new device:

```
./tty_fake -s /tmp/fakebus /dev/fakedevice1
```

this command will output the pseudo-terminal just created to use as interface in the application, for example `/dev/pts/#`.

As for the fake bus, this new interface, let's say`/dev/pts/4`, will expire when the `tty_fake` process is killed.

**Detaching the application from the interface won't cause its closing.**

