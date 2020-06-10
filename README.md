# nas_fan_control
collection of scripts to control fan speed on NAS boxes

This is a fork of Kevin Horton's repository:

	https://github.com/khorton/nas_fan_control

I made extensive changes to PID_fan_control.pl:

* added global $script_mode variable which controls whether the script is controlling fans attached to a SuperMicro motherboard with its fan "zones", or to an ASRock-Rack motherboard, which has no zones.
* added capability to become a UNIX daemon (depends on Proc::Daemon)
* declaring global and local variables explicitly
* reformatting comments and some code to get it to fit on a screen less than 219 characters wide
* getting rid of all the calls to superfluous external programs such as [e]grep, awk, sed and using pure Perl
* simplifying where possible (for example removing all the newlines from the calls to dprint() and adding one in the dprint() function)
* in general, getting it to pass a basic syntax check with "use strict; use warnings;"
* a few bug fixes

I added dependencies on two Perl packages:

* IPC::RUN
* Proc::Daemon

Which means these Perl packages must be installed directly on the FreeNAS OS (if you are running the script there), which is not supported. This requires changing the repo control files in /usr/local/etc/pkg/repos/. If this bothers you, it is possible to accomplish these tasks (daemonizing, running external processes) without these dependencies. Please submit a request and I may find the time to remove the dependencies.

If $script_mode is set to "asrock", then the "zones" are defined by @asrock_zones, and ipmi command to set the duty-cycle is:
ipmitool raw 0x3a 0x01 <FAN1-duty-cycle> .. <FAN6-duty-cycle> <filler> <filler>

@asrock_zones is a data-structure with the following format:
my @asrock_zones = (
	{ <FAN-NAME> => { <FAN-ENTRY> }, [<FAN-NAME> => { <FAN-ENTRY> } }, # zone-0
	{ <FAN-NAME> => { <FAN-ENTRY> }, [<FAN-NAME> => { <FAN-ENTRY> } }, # zone-1
);
where:
	<FAN-NAME> is arbitrary. I used "FAN1" .. "FAN6"
	<FAN-ENTRY> is a hashref with two keys, "index" and optionally "factor"
		index points to the fan position in the ipmitool command
		factor is a floating value that modifies the duty-cycle value of the
			PID controller (will be limited to 100%)

This data-structure should ideally find its way into a config file.

My case has a "fan wall" between the front section, where the HDs reside, and the rear section, where the mobo and PSU reside. The fan wall has three identical fans (120mm Corsair Maglevs). These are connected to FAN2, FAN3 and FAN4. The rear of the case has two 80mm Be Quiet! fans connected to FAN5 and FAN6. Because I think the case fans should spin faster than the ones in the fan wall, I used factor 1.1 (10% faster) on them.

I did not touch any of the PID controller logic.

This is now successfully controlling the fans on my ASRock-Rack X470D4U2-2T quite satisfactorily.

There is more cleanup/restructuring to come...

- Rob Urban

The following is the content from Kevin's README.md

PID_fan_control.pl - Perl fan control script based on the hybrid fan control script created by @Stux, and posted at:
https://forums.freenas.org/index.php?threads/script-hybrid-cpu-hd-fan-zone-controller.46159/ .  @Stux's script was modified by replacing his fan control loop with a PID controller.  This version of the script was settings and gains used by the author on a Norco RPC-4224, with the following fans:

*  3 x Noctua NF-F12 PWM 120mm fans: hard drive wall fans replaced with .  
*  2 x Noctua NF-A8 PWM 80mm fans: chassis exit fans.  
*  1 x Noctua NH-U9DX I4: CPU cooler.

The hard drive fans are connected to fan headers assigned to the hard drive temperature control portion of the script.  The chassis exit fans and the CPU cooler are connected to fan headers assigned to the CPU temperature control portion of the script.

See the scripts for more info and commentary.

Discussion on the FreeNAS forums: https://forums.freenas.org/index.php?threads/pid-fan-controller-perl-script.50908/
