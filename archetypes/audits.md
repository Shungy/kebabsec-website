---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
toc: true
---

## Executive Summary

## Findings

### [SEVERITY] Title

Impact and overview.

#### Proof of Concept

```solidity {linenos=table,linenostart=30,hl_lines=[2,"4-7"]}
function configureDependencies() external override returns (Keycode[] memory dependencies) {
    dependencies = new Keycode[](1);
    dependencies[0] = toKeycode("VOTES");

    VOTES = VOTESv1(getModuleAddress(dependencies[0]));

    gOHM.approve(address(VOTES), type(uint256).max);
}
```

#### Recommendation
