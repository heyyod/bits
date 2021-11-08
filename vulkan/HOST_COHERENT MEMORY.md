# HOST_COHERENT MEMORY

When the buffer is NOT host coherent we have to flush/invalidate the caches.

Flushing/invalidating caches is needed because when the device writes some data to RAM, the CPU caches might not be up-to-date, and you get an old value when you read from the data.

Similarly, writes might be stuck in a write cache, so the device can't access them even though you technically wrote them. Flushing caches writes all values to actual memory, and invalidating caches ensures you can't read old values in caches that the device has since updated. With coherent memory, this isn't necessary because caching is managed w. r. t. device accesses too.

imagine for example uint32_t x[2];
cpu writes x[0] = 1;
gpu writes x[1] = 2;

if x lies within a single "non coherent atom", your program is broken at this point
err, this is assuming caches are also at play
in that case, either cpu will write out entire non coherent atom back (and effectively overwrite gpu's write to x[1]), or gpu will
either way, nothing good happens
now, if there's coherence, there's no such problem anymore

if you wrote from cpu and you want gpu to access some memory, you "flush"
if you wrote from gpu and you intend to access from cpu, you use "invalidate"
referring specifically to these two functions
