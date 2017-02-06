---
title: Perlin Noise
date: 2017-01-10 19:24:09
tags: [Java, Util]
---

普通随机数算法生成的随机数真的很随意，不能做到在某个序列内是升序或者降序的，如果要生成某个范围内一序列的升序或者降序的随机数，例如用于游戏中生成随机地图，渲染海洋等，可以使用 Noise  随机数生成算法，Flow Noise 是 Noise 算法的 Java 实现，其 github 地址为 [https://github.com/flow/noise#documentation](https://github.com/flow/noise#documentation)

> Noise generation library for Java, based on the libnoise C++ library. It is used to generate coherent noise, a type of smoothly-changing noise. It can also generate Perlin noise, ridged multifractal noise, and other types of coherent noise. [https://flowpowered.com/noise](https://flowpowered.com/noise) <!--more-->

## Gradle 依赖

```groovy
compile 'com.flowpowered:flow-noise:1.0.0'
```

## 测试

```java
import com.flowpowered.noise.NoiseQuality;
import com.flowpowered.noise.module.source.Perlin;

public class Noise {
    public static void main(String[] args) throws Exception {
        Perlin noise = new Perlin();
        noise.setNoiseQuality(NoiseQuality.BEST);

        double x = 0;
        for (int i = 0; i < 100; i++) {
            System.out.println(noise.getValue(x, 0, 0));
            x += 0.2; // x 的差值越大，随机数之间的差值也越大
        }
    }
}
```

输出:

```
0.0
-0.32461191667440004
-0.21562247324921602
-0.24045666577702365
0.08154942540182364
0.0
0.017760199481471912
0.07624406124044805
0.09420645897823994
0.3106714033895679
2.0356525443077088E-16
-0.5009738816233593
-0.5280269381836802
-0.08401516305952028
-0.0963942882222088
1.4972738959784236E-15
0.2220479107193613
0.4853802991390389
0.10574431637968457
0.24034578731172135
2.058906360957735E-15
-0.37327679101936373
0.13443128051712325
0.2044925554560296
0.24911839574371034
-6.323059913881934E-15
-0.11428265826071174
-0.1556519205386192
0.03624678852102316
0.09369426584192217
5.48520247889428E-15
-0.10981229723004368
0.07450831583660719
0.1874437317384376
0.4502187874492507
-3.015403592598887E-14
-0.6337037690724758
-0.5491793529709761
-0.029993830662126027
0.4045362208994956
-2.8010581871740217E-14
-0.0672840589676809
0.1265643203913621
-0.1359116959702379
0.16337980840319843
0.0
-0.6957311878065602
-0.5816246121859834
-0.3056587388310477
0.33203871305418214
-1.1401320799109271E-14
0.07583105593798484
0.5880399795814963
0.6221210894496275
0.5331957522801097
-1.1189377119080746E-14
-0.45889112933202025
-0.4873910226824587
0.08115698823824825
0.26824352832699966
3.20811328435866E-14
-0.2677929777421113
-0.5723440541260215
-0.09600853545427149
-0.16037053682661212
-1.8215002342003515E-15
0.18576428417607369
0.2291702568342356
0.4585356265048988
0.5902431697298186
4.291813539225586E-14
-0.3177010886241441
-0.44926368503407826
0.10899072323686518
0.22870032101347187
2.8381001015986882E-15
-0.4219754122814422
-0.33899021205891255
0.22157360632138023
0.2770001468648504
9.257461385914213E-14
-0.17037175480717265
-0.30889256228653045
-0.8330496382101313
-0.47127551743719076
-1.6003065070435696E-13
0.6000432296379951
0.4131921907177562
-0.10313697596534846
-0.15818511143040162
6.108196700438384E-14
0.13794012783972323
0.3898001391189491
0.25297850534722144
0.04878544281038039
-1.4850726444137755E-14
0.06003376532714952
0.4473940276033668
0.051663826772296056
0.1221529711843983
```


