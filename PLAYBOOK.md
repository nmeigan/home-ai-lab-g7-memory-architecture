# Playbook — Building the Home AI Lab

Ordered steps, as actually executed. Read `ARCHITECTURE.md` first if you want the reasoning before the commands.

**Warning:** the disk operations below (`diskpart`, partition deletion) are destructive if run against the wrong disk. Always run `list volume` and confirm which physical disk maps to which drive letter before any `delete partition` command.

## 1. Recovering a cloned SSD that lost its boot partition

If you clone a drive and the boot files stay orphaned (e.g. after deleting what looks like a leftover/"ghost" partition that was actually the active EFI boot partition):

1. Open Disk Management, identify the drive letter of the new drive's OS partition and its FAT32 EFI partition (~100MB).
2. From an elevated Command Prompt, rebuild the boot files:
   ```
   bcdboot <OS-partition-letter>:\Windows /s <EFI-partition-letter>: /f UEFI
   ```
3. If the new drive is connected externally (USB/Thunderbolt), Windows will refuse to boot from it natively — you must physically install it in the internal slot first.
4. After the physical swap, the new drive typically reclaims the `C:` letter automatically. Confirm with:
   ```
   diskpart
   list volume
   ```

## 2. Cleaning up leftover partitions from a clone

Only after confirming with `list volume` which disk is the live boot disk vs. the old drive:

```
diskpart
list disk
select disk <N>          # the OLD/secondary disk, verified via list volume first
list partition
select partition <X>      # the ~100MB EFI remnant
delete partition override
select partition <Y>      # the recovery partition, if present
delete partition override
exit
```

## 3. Windows page file (separate from WSL2's own swap)

Size the page file against your actual NVMe throughput, not the generic "1.5x RAM" rule (which assumes a much slower disk):

1. `sysdm.cpl` → Advanced → Performance → Settings → Advanced → Virtual Memory → Change
2. Uncheck "Automatically manage paging file size for all drives"
3. Select the fast NVMe drive, choose Custom size
4. Initial size: `16384` MB (16GB) — Maximum size: `49152` MB (48GB)
5. Click **Set** (not just OK), then OK through all dialogs, then restart

## 4. WSL2 memory and swap — the actual lever

By default WSL2 takes 50% of physical RAM and a small, fixed swap. For loading 20B+ parameter models, override this explicitly.

Create `%USERPROFILE%\.wslconfig`:

```ini
[wsl2]
memory=26GB
swap=32GB
processors=12
pageReporting=true
```

(Omit an explicit `swapFile=` path — letting Windows build the swap disk in its default location avoids a common quoting/escaping failure with a custom path.)

Apply it:
```
wsl --shutdown
```

Verify inside the WSL2 terminal:
```
free -h --giga
```
Expected: `Mem: 26G` total, `Swap: 32G` total. If Swap shows `0B`, the `.wslconfig` wasn't picked up — double-check the file has no `.txt` extension and re-run `wsl --shutdown`.

## 5. Keeping model weights out of cloud sync

Local model files (2GB–23GB+ each) inside a OneDrive-synced user profile will cause continuous re-upload attempts and pin your CPU/fans. Move them out:

1. Quit the sync client and the model server before moving files.
2. Move the models folder to a non-synced drive/folder.
3. Set a `OLLAMA_MODELS` (or equivalent) environment variable to the new path.
4. Fully quit and relaunch the model server so it picks up the new path (a background service will not see the new env var until restarted).
5. Verify with the model list command that the models registered from the new location, and confirm via the app's own log file that it's reading the new path.

## 6. Bridging a Windows-hosted model server to a WSL2 Linux shell

If you run the model server natively on Windows (for native GPU acceleration) but want to drive it from a WSL2 Linux terminal:

```powershell
# In Windows PowerShell (not WSL2):
[Environment]::SetEnvironmentVariable("OLLAMA_HOST", "0.0.0.0:11434", "User")
# Restart the model server app after setting this.
```

```bash
# In WSL2 Linux terminal:
export OLLAMA_HOST=$(ip route | grep default | awk '{print $3}'):11434
curl http://localhost:11434   # sanity check via the gateway IP, not "localhost"
```

## 7. Capping GPU offload against measured usage, not a guess

Never assume a fixed VRAM budget — measure it. With a model loaded, watch actual usage:

```
nvidia-smi
```

Tune the inference engine's GPU-offload/layer-count setting against that measured number, not a guessed layer count. On a 4GB-VRAM card, ~2.5GB was the stable usable ceiling here; the remaining layers spill to CPU/RAM. If GPU offload proves unstable for a given model at any setting, fall back to CPU-only (`num_gpu=0` in Ollama terms) rather than forcing an unstable configuration — an unstable GPU offload crashes mid-generation, which is worse than a slower but reliable CPU-only run.

## 8. Choosing a model tier

See `RESULTS.md` for the measured speed/quality tradeoffs. As a starting rule of thumb on a 26GB physical + 32GB swap pool:
- **14B class** (~9GB): fits fully in RAM, fast, everyday chat/reasoning
- **32B class** (~20GB): fits fully in RAM, the genuine sweet spot for coding/reasoning
- **70B class, 4-bit** (~42–45GB): spills deep into swap, loads without crashing, but token generation is glacial — treat as a ceiling test, not daily use
