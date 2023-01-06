---
title: Metaplex阅读理解
date: 2022-04-15T01:55:13.000Z
draft: true
toc: false
images: null
tags:
  - untagged
---

# Metaplex阅读 理解

## 代码内容

### mint_nft

1. 检查candy_machine是否可以工作
2. 检查是否已经end_settings
3. gatekeeper
4. 如果有whitelist，进行whitelist检查
5. candy_machine.token_mint 有数据，就发起 spl_token_transfer. 没有则invoke
6. 获取creator列表，准备metadata_info和master_edition_info
7. 执行3次invoke_signed，instruction（指令）是 create_metadata_accounts_v2 和 create_master_edition_v3, update_metadata_accounts_v2
8. 遍历num_instructions,检查是否有可以交易

### set_collection_during_mint

### update_candy_machine

### add_config_lines

### initialize_candy_machine

### set_collection

### remove_collection

### update_authority

### withdraw_funds

## 代码执行

TRANSACTION                                      | SIGNATURE | AGE         | INSTRUCTION        | PROGRAM                      | RESULT
------------------------------------------------ | --------- | ----------- | ------------------ | ---------------------------- | -------
4yH65MvVTirNrkQbRWF8ib8T3kCcVFb9MWCsMGEYzfN5AXGh | ...       | 19 days ago | Initialize Mint    | Token Program                | Success |
4yH65MvVTirNrkQbRWF8ib8T3kCcVFb9MWCsMGEYzfN5AXGh | ...       | 19 days ago | Initialize Account | Token Program                | Success |
4yH65MvVTirNrkQbRWF8ib8T3kCcVFb9MWCsMGEYzfN5AXGh | ...       | 19 days ago | Mint To            | Token Program                | Success |
4yH65MvVTirNrkQbRWF8ib8T3kCcVFb9MWCsMGEYzfN5AXGh | ...       | 19 days ago | Unknown (Inner)    | NFT Candy Machine Program V2 | Success |
4yH65MvVTirNrkQbRWF8ib8T3kCcVFb9MWCsMGEYzfN5AXGh | ...       | 19 days ago | Unknown (Inner)    | Token Metadata Program       | Success |
4yH65MvVTirNrkQbRWF8ib8T3kCcVFb9MWCsMGEYzfN5AXGh | ...       | 19 days ago | Set Authority      | Token Program                | Success |
4yH65MvVTirNrkQbRWF8ib8T3kCcVFb9MWCsMGEYzfN5AXGh | ...       | 19 days ago | Set Authority      | Token Program                | Success |
