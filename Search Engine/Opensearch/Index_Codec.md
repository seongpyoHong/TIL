Opensearch 2.19부터 Zstandard compression algorithm을 기반으로 한 codec을 지원한다. 이는 CPU 사용률과 indexing, search 성능이 default codec과 비교했을 때 좋아진다. 아래 두 옵션 중 `zstd_no_dict`를 사용했다. 

- zstd (OpenSearch 2.9 and later) – This codec provides significant compression comparable to the best_compression codec with reasonable CPU usage and improved indexing and search performance compared to the default codec.
- zstd_no_dict (OpenSearch 2.9 and later) – This codec is similar to zstd but excludes the dictionary compression feature. It provides faster indexing and search operations compared to zstd at the expense of a slightly larger index size.

### Reference
https://opensearch.org/docs/latest/im-plugin/index-codecs/