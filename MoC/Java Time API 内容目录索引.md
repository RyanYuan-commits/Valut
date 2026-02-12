---
type: MOC
banner: Assets/Banner/pexels-faikackmerd-1025469.jpg
---

```mermaid
graph LR
    %% 节点定义
    subgraph MachineTime [机器时间]
        Instant((Instant))
    end

    subgraph LocalTimeGroup [本地时间 - 无时区]
        LD[LocalDate]
        LT[LocalTime]
        LDT[LocalDateTime]
    end

    subgraph ZonedTimeGroup [带时区时间]
        ZDT[ZonedDateTime]
        ODT[OffsetDateTime]
    end

    %% 转化关系
    
    %% Instant 与 Zoned/Offset
    Instant -- "atZone(ZoneId)" --> ZDT
    Instant -- "atOffset(ZoneOffset)" --> ODT
    ZDT -- "toInstant()" --> Instant
    ODT -- "toInstant()" --> Instant

    %% Local 与 Zoned/Offset
    LDT -- "atZone(ZoneId)" --> ZDT
    LDT -- "atOffset(ZoneOffset)" --> ODT
    ZDT -- "toLocalDateTime()" --> LDT
    ODT -- "toLocalDateTime()" --> LDT

    %% Local 内部互转
    LD -- "atTime(LocalTime)" --> LDT
    LT -- "atDate(LocalDate)" --> LDT
    LDT -- "toLocalDate()" --> LD
    LDT -- "toLocalTime()" --> LT

    %% ZDT 与 ODT 互转
    ZDT -- "toOffsetDateTime()" --> ODT
    ODT -- "toZonedDateTime()" --> ZDT

    %% 样式美化
    style Instant fill:#f9f,stroke:#333,stroke-width:2px
    style LDT fill:#bbf,stroke:#333,stroke-width:2px
    style ZDT fill:#bfb,stroke:#333,stroke-width:2px
    style ODT fill:#dfd,stroke:#333,stroke-width:1px
```

# 🔗 概念之间的关系

这些卡片解释了 "为什么" 和 "底层逻辑"

[[JSR-310 设计哲学 --- 不可变性与线程安全]]

[[机器时间 vs 人类时间]]

这些卡片是一些原子知识

[[Java Instant —— 时间轴上的绝对点]]

[[LocalXXX —— 本地时间的语境]]

[[ZonedDateTime vs OffsetDateTime —— 时区处理]]

[[Duration vs Period --- 时间量度的维度]]