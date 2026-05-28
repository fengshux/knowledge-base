---
title: S3 CopyObject vs UploadPartCopy 区别
date: 2026-05-15 15:00
tags: [AWS, S3, 对象存储, CopyObject, UploadPartCopy, 多段上传]
category: devops
---

# S3 CopyObject vs UploadPartCopy 区别

## 核心概念

- **CopyObject**：S3 的原子性单次复制操作，将源对象完整复制到目标位置，上限 5GB。
- **UploadPartCopy**：多段上传（Multipart Upload）中的子操作，将源对象的一部分作为一段 Part 复制到目标多段上传中，配合 Initiate/Complete 实现大对象复制，上限 5TB。

## 使用场景

| 场景 | 推荐操作 |
|------|---------|
| 复制 ≤ 5GB 的小文件 | CopyObject |
| 复制 > 5GB 的大文件 | UploadPartCopy |
| 需要并行加速复制 | UploadPartCopy |
| 只复制源对象的部分内容 | UploadPartCopy（指定 byte range） |
| 更新对象元数据或存储级别 | CopyObject |
| 断点续传式复制 | UploadPartCopy |

## 核心区别一览

| 对比维度 | CopyObject | UploadPartCopy |
|---------|-----------|---------------|
| 最大对象大小 | 5GB | 5TB |
| 请求次数 | 1 次 | 至少 3 步（Init + N × Part + Complete） |
| 并行能力 | ❌ 不支持 | ✅ 支持多 Part 并行 |
| 原子性 | ✅ 原子操作 | ❌ 需 Complete 后才整体生效 |
| 源范围选择 | ❌ 只能复制整个对象 | ✅ 可指定 byte range |
| 元数据处理 | 复制时可指定 | 在 Initiate 阶段指定 |
| 失败恢复 | 全部重试 | 只重试失败的 Part |

## 代码示例

### CopyObject（≤ 5GB）

```python
import boto3

s3 = boto3.client('s3')
s3.copy_object(
    Bucket='target-bucket',
    Key='target-key',
    CopySource={'Bucket': 'source-bucket', 'Key': 'source-key'}
)
```

### UploadPartCopy（> 5GB / 并行加速）

```python
import boto3

s3 = boto3.client('s3')

# 1. 初始化多段上传
response = s3.create_multipart_upload(
    Bucket='target-bucket', Key='target-key'
)
upload_id = response['UploadId']

# 2. 分段复制
parts = []
part_size = 8 * 1024 * 1024  # 8MB per part
source_size = 100 * 1024 * 1024  # 100MB source

for i in range(1, (source_size // part_size) + 2):
    start = (i - 1) * part_size
    end = min(i * part_size - 1, source_size - 1)
    
    part = s3.upload_part_copy(
        Bucket='target-bucket',
        Key='target-key',
        PartNumber=i,
        UploadId=upload_id,
        CopySource={'Bucket': 'source-bucket', 'Key': 'source-key'},
        CopySourceRange=f'bytes={start}-{end}'
    )
    parts.append({'PartNumber': i, 'ETag': part['CopyPartResult']['ETag']})

# 3. 完成多段上传
s3.complete_multipart_upload(
    Bucket='target-bucket',
    Key='target-key',
    UploadId=upload_id,
    MultipartUpload={'Parts': parts}
)
```

## 注意事项

- CopyObject 底层其实也是多段复制，但由 S3 服务端自动完成，对客户端透明
- UploadPartCopy 把分段控制权交给客户端，可控制并发度、Part 大小和重试策略
- 同区域复制走服务端内部网络，不产生数据传输费用；跨区域复制会产生数据传输费
- UploadPartCopy 的 Part 大小除最后一段外，必须 ≥ 5MB（小于 5MB 的 Part 只有最后一段允许）
- 如果中途失败，可以调用 AbortMultipartUpload 清理未完成的上传

## 最佳实践

- 小文件（≤ 5GB）无脑用 CopyObject，简单可靠
- 大文件（> 5GB）必须用 UploadPartCopy
- 大文件建议 Part 大小设为 8MB~64MB，并发数 5~10 个线程
- 对于超大文件（> 100GB），考虑使用 S3 Batch Operations 或 AWS CLI 的 `aws s3 cp`（自动处理多段逻辑）

## 参考资料

- [AWS S3 CopyObject API 文档](https://docs.aws.amazon.com/AmazonS3/latest/API/API_CopyObject.html)
- [AWS S3 UploadPartCopy API 文档](https://docs.aws.amazon.com/AmazonS3/latest/API/API_UploadPartCopy.html)
- [AWS S3 Multipart Upload 概述](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html)

---

*最后更新：2026-05-15*