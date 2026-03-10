# 经验文档 - 2026-03-10

**生成时间**: 2026-03-10T14:30:31.832144

---

## 💡 经验教训


### 经验 1

Follow skill-creator best practices. Keep SKILL.md under 500 lines. Use references/ for detailed docs.


### 经验 2

CRITICAL: After installing NVIDIA driver, must disable nouveau and reboot. Check with 'nvidia-smi' and 'lspci -k | grep nvidia'. This is a common issue that can save hours of debugging.


### 经验 3

User experience is paramount. Always provide immediate feedback. Use process poll with short timeouts. Never let user wait in silence for more than 10-15 seconds.


## 🔍 相关错误

- error_fix: 
**Details**:
```json
{
  "machine": "192.168.136.128",
  "gpu": "RTX 4080",
  "error": "CUDA not av...
