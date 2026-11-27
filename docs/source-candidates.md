# Public kernel source candidates

These are leads for source investigation, not validated drop-in kernels for the owner's firmware.

## Candidate A: BS1388/Kernel_Samsung_a346E
- Repository: https://github.com/BS1388/Kernel_Samsung_a346E
- Repository README identifies it as kernel source for Samsung Galaxy A34 5G, a346E.
- Its README also links a separate AnyKernel3 package. That package is not being treated as safe or recommended for installation.

## Candidate B: UN1CA/kernel_samsung_a34x
- Repository: https://github.com/UN1CA/kernel_samsung_a34x
- Search result indicated a kernel 6.6 / Android 16-era branch. Exact firmware and device-tree match remains unverified.

## Verification still required
- Confirm source commit/branch maps to firmware A346EXXSGFZJ4 and bootloader binary G.
- Confirm stock configuration and vendor module compatibility.
- Identify touchscreen IC and driver from this exact variant.
- Locate device-tree properties and controller firmware definitions.
- Establish controller-native maximum report rate from primary documentation.

Do not infer a matching kernel solely from the model name or the 6.6 version number.
