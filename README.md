# 欧西1847 · 数字自证

一个人、一台笔记本、零成本，把一颗芯片从 RTL 走到 GDS，然后要求页面上的每个数字都能指到证据文件的某一行。

- 项目站点：https://1ouxilisi.github.io/sha256-openlane-tapeout/
- 仓库：https://github.com/1ouxilisi/sha256-openlane-tapeout
- 作品集（11 页 PDF，含整页版图）：https://1ouxilisi.github.io/sha256-openlane-tapeout/portfolio.pdf

## 这颗芯片

SHA-256 硬件加速器，SkyWater sky130A 工艺、OpenLane 1.1.1 全开源流程，单 die 周期 18 ns。

同一份 GDS（40,666,588 字节，sha256 `89cda760…`）上完成的复核：

<img src="https://1ouxilisi.github.io/sha256-openlane-tapeout/images/sha256_antclean_layout.png" alt="天线 0/0、补齐 fill 这一版 GDS 的金属层渲染" width="100%">

上面就是那份 GDS 自己的渲染：展平后 3,021,383 个多边形，bbox 0,0 – 640,640 µm，只画 li1 与 m1–m5 六层（`gdspy` 展平 + `matplotlib` 上色，不是查看器截图）。


- KLayout signoff DRC：218 个规则类、0 条；正对照同 deck 立刻报 1 条，证明不是空跑
- 金属密度：li1 / m1 / m2 / m3 / m4 / m5 六层空孔率 0.4991–0.9716，全部在阈值之上、0 条
- LVS：Magic 从 GDS 提取的晶体管网表对 OpenROAD 写出的网表，`Circuits match uniquely`，两侧 34,289 个实例、94 个端口
- 天线：复检先报 39 net / 49 pin（不是 0），随后同一份布线后 ODB 上三轮压到 0 / 0，另起进程独立复检仍 0 / 0
- GDS XOR：两条独立 stream-out 路径（Magic 出图 / KLayout 回填单元）flatten 后逐层异或，`TOTAL_DIFF=0`

## 边界也写在这里

- 天线归零那一版没有重跑 RCX + 三 corner STA 与 DRV，所以时序结论属于上一版（ctsfix3）原版，不能顺延；
- ctsfix3 那版只做到「STA 三 corner 全 MET」：那一次 flow 在第 36 步假死，LVS / 天线 / DRC 与 DRV 五列指标都是 `-1`（没跑，不是 0），这三项是后来单独补跑的；真正跑过 DRV 的派生版还留 90 条 max slew（最差 1.04 ns，限 0.75 ns）；
- 本版 LVS 两侧 34,289 实例与上一版 34,182 相差 +107，未逐条追到单元级，照写在页面上而不是抹平。

所以这里没有「signoff 通过」，只有分项列出、每项都能追到行号的结论。

## 怎么核

`docs/evidence/` 下每一份原文都带字节数、行数、sha256，页面里引用结论时给出具体文件与行号；被推翻的旧写法保留在页面上，用「此处更正」标出，不覆盖。

流程里踩过的坑、三次公开撤回的归因、以及「0 条不等于空跑」的正对照做法，写在技术博客与站点里。
