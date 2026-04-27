# 方案5实现细节 - 完整指南

## 架构设计

### 系统架构图

```
用户输入
    ↓
参数解析（--fast / --deep）
    ↓
缓存检查
    ├─ 缓存命中 → 返回缓存结果
    └─ 缓存未命中 → 继续处理
    ↓
fetch → score → review
    ↓
concepts-extract
    ├─ 检查缓存中的概念
    ├─ 识别新概念
    └─ 更新缓存
    ↓
notes
    ↓
输出结果 + 更新缓存
```

---

## Phase 1：缓存系统实现

### 1.1 缓存数据结构

```python
# cache.json 结构
{
  "version": "1.0",
  "last_updated": "2026-04-27T10:30:00Z",
  "concepts_cache": {
    "seed_ai_mental_health_a1b2c3": {
      "id": "seed_001",
      "canonical_id": "seed_ai_mental_health_a1b2c3",
      "name": "AI×心理健康",
      "keywords": ["AI", "mental health", "therapy"],
      "cached_at": "2026-04-27",
      "hit_count": 5,
      "last_hit": "2026-04-27T15:30:00Z"
    },
    "auto_transformer_model_g7h8i9": {
      "id": "auto_001",
      "canonical_id": "auto_transformer_model_g7h8i9",
      "name": "Transformer模型",
      "keywords": ["transformer", "attention"],
      "confidence": 0.92,
      "cached_at": "2026-04-27",
      "hit_count": 3,
      "last_hit": "2026-04-27T14:00:00Z"
    }
  },
  "paper_cache": {
    "paper_001_hash": {
      "title": "LLM-based Therapy Assistant",
      "abstract_hash": "abc123def456",
      "concepts": ["seed_001", "auto_001"],
      "cached_at": "2026-04-27",
      "hit_count": 2
    }
  },
  "search_cache": {
    "AI_mental_health_hash": {
      "keywords": ["AI", "mental health"],
      "results": ["paper_001", "paper_002"],
      "cached_at": "2026-04-27",
      "hit_count": 3,
      "ttl": 7  # 7天过期
    }
  }
}
```

### 1.2 缓存管理类

```python
import json
import hashlib
from datetime import datetime, timedelta
from pathlib import Path

class ConceptCache:
    def __init__(self, cache_dir="~/Desktop/playmore"):
        self.cache_dir = Path(cache_dir).expanduser()
        self.cache_file = self.cache_dir / "cache.json"
        self.cache = self._load_cache()
    
    def _load_cache(self):
        """加载缓存"""
        if self.cache_file.exists():
            with open(self.cache_file, 'r') as f:
                return json.load(f)
        return {
            "version": "1.0",
            "last_updated": datetime.now().isoformat(),
            "concepts_cache": {},
            "paper_cache": {},
            "search_cache": {}
        }
    
    def _save_cache(self):
        """保存缓存"""
        self.cache["last_updated"] = datetime.now().isoformat()
        with open(self.cache_file, 'w') as f:
            json.dump(self.cache, f, indent=2)
    
    def get_concept(self, canonical_id):
        """获取缓存的概念"""
        if canonical_id in self.cache["concepts_cache"]:
            concept = self.cache["concepts_cache"][canonical_id]
            # 更新hit_count和last_hit
            concept["hit_count"] = concept.get("hit_count", 0) + 1
            concept["last_hit"] = datetime.now().isoformat()
            self._save_cache()
            return concept
        return None
    
    def set_concept(self, canonical_id, concept):
        """缓存概念"""
        concept["cached_at"] = datetime.now().isoformat()
        concept["hit_count"] = 0
        self.cache["concepts_cache"][canonical_id] = concept
        self._save_cache()
    
    def get_paper_concepts(self, paper_hash):
        """获取缓存的论文概念"""
        if paper_hash in self.cache["paper_cache"]:
            paper = self.cache["paper_cache"][paper_hash]
            paper["hit_count"] = paper.get("hit_count", 0) + 1
            self._save_cache()
            return paper.get("concepts", [])
        return None
    
    def set_paper_concepts(self, paper_hash, paper_info, concepts):
        """缓存论文概念"""
        self.cache["paper_cache"][paper_hash] = {
            "title": paper_info.get("title"),
            "abstract_hash": self._hash_text(paper_info.get("abstract", "")),
            "concepts": concepts,
            "cached_at": datetime.now().isoformat(),
            "hit_count": 0
        }
        self._save_cache()
    
    def get_search_results(self, keywords):
        """获取缓存的搜索结果"""
        search_hash = self._hash_text("_".join(keywords))
        if search_hash in self.cache["search_cache"]:
            search = self.cache["search_cache"][search_hash]
            # 检查TTL
            cached_at = datetime.fromisoformat(search["cached_at"])
            ttl_days = search.get("ttl", 7)
            if datetime.now() - cached_at < timedelta(days=ttl_days):
                search["hit_count"] = search.get("hit_count", 0) + 1
                self._save_cache()
                return search.get("results", [])
        return None
    
    def set_search_results(self, keywords, results, ttl=7):
        """缓存搜索结果"""
        search_hash = self._hash_text("_".join(keywords))
        self.cache["search_cache"][search_hash] = {
            "keywords": keywords,
            "results": results,
            "cached_at": datetime.now().isoformat(),
            "hit_count": 0,
            "ttl": ttl
        }
        self._save_cache()
    
    def clear_expired(self):
        """清理过期缓存"""
        now = datetime.now()
        
        # 清理搜索缓存
        expired_keys = []
        for key, search in self.cache["search_cache"].items():
            cached_at = datetime.fromisoformat(search["cached_at"])
            ttl_days = search.get("ttl", 7)
            if now - cached_at > timedelta(days=ttl_days):
                expired_keys.append(key)
        
        for key in expired_keys:
            del self.cache["search_cache"][key]
        
        if expired_keys:
            self._save_cache()
        
        return len(expired_keys)
    
    def get_stats(self):
        """获取缓存统计"""
        return {
            "concepts_count": len(self.cache["concepts_cache"]),
            "papers_count": len(self.cache["paper_cache"]),
            "searches_count": len(self.cache["search_cache"]),
            "total_hits": sum(
                c.get("hit_count", 0) 
                for c in self.cache["concepts_cache"].values()
            ) + sum(
                p.get("hit_count", 0) 
                for p in self.cache["paper_cache"].values()
            ),
            "last_updated": self.cache["last_updated"]
        }
    
    @staticmethod
    def _hash_text(text):
        """生成文本hash"""
        return hashlib.md5(text.encode()).hexdigest()[:8]
```

### 1.3 缓存集成到概念提取

```python
def extract_concepts_with_cache(papers, mode="standard"):
    """
    使用缓存的概念提取
    
    mode: "fast" / "standard" / "deep"
    """
    cache = ConceptCache()
    
    # 清理过期缓存
    expired = cache.clear_expired()
    if expired > 0:
        print(f"清理了 {expired} 个过期缓存")
    
    all_concepts = []
    papers_to_process = []
    
    # Step 1: 检查缓存
    for paper in papers:
        paper_hash = cache._hash_text(paper["title"] + paper.get("abstract", ""))
        cached_concepts = cache.get_paper_concepts(paper_hash)
        
        if cached_concepts:
            print(f"✓ 使用缓存：{paper['title']}")
            all_concepts.extend(cached_concepts)
        else:
            print(f"✗ 需要处理：{paper['title']}")
            papers_to_process.append((paper, paper_hash))
    
    # Step 2: 处理未缓存的论文
    if papers_to_process:
        # 计算复杂度
        simple_papers = []
        complex_papers = []
        
        for paper, paper_hash in papers_to_process:
            complexity = calculate_complexity(paper)
            if complexity > 0.6:
                complex_papers.append((paper, paper_hash))
            else:
                simple_papers.append((paper, paper_hash))
        
        # 批量处理简单论文
        if simple_papers:
            batch_size = 5
            for i in range(0, len(simple_papers), batch_size):
                batch = [p[0] for p in simple_papers[i:i+batch_size]]
                concepts = call_claude_batch(batch)
                
                # 缓存结果
                for j, (paper, paper_hash) in enumerate(simple_papers[i:i+batch_size]):
                    cache.set_paper_concepts(paper_hash, paper, concepts[j])
                    all_concepts.extend(concepts[j])
        
        # 逐篇处理复杂论文
        for paper, paper_hash in complex_papers:
            concepts = call_claude_single(paper)
            cache.set_paper_concepts(paper_hash, paper, concepts)
            all_concepts.extend(concepts)
    
    # Step 3: 缓存概念
    for concept in all_concepts:
        if "canonical_id" in concept:
            cache.set_concept(concept["canonical_id"], concept)
    
    # Step 4: 显示统计
    stats = cache.get_stats()
    print(f"\n缓存统计：")
    print(f"  概念数：{stats['concepts_count']}")
    print(f"  论文数：{stats['papers_count']}")
    print(f"  搜索数：{stats['searches_count']}")
    print(f"  总命中：{stats['total_hits']}")
    
    return all_concepts
```

---

## Phase 2：分层模式实现

### 2.1 模式定义

```python
class ProcessingMode:
    """处理模式定义"""
    
    FAST = {
        "name": "fast",
        "description": "快速模式 - 只做基础分析",
        "steps": ["fetch", "score"],
        "skip_steps": ["review", "concepts", "notes"],
        "token_estimate": "3000-4000",
        "quality": "⭐⭐⭐",
        "time_estimate": "30秒"
    }
    
    STANDARD = {
        "name": "standard",
        "description": "标准模式 - 完整分析（推荐）",
        "steps": ["fetch", "score", "review", "concepts", "notes"],
        "skip_steps": [],
        "token_estimate": "9640-13000",
        "quality": "⭐⭐⭐⭐⭐",
        "time_estimate": "2-3分钟"
    }
    
    DEEP = {
        "name": "deep",
        "description": "深度模式 - 最全面的分析",
        "steps": ["fetch", "score", "review", "concepts", "organize", "notes"],
        "skip_steps": [],
        "token_estimate": "12000-16000",
        "quality": "⭐⭐⭐⭐⭐⭐",
        "time_estimate": "3-5分钟"
    }
    
    @classmethod
    def get_mode(cls, mode_name):
        """获取模式配置"""
        modes = {
            "fast": cls.FAST,
            "standard": cls.STANDARD,
            "deep": cls.DEEP
        }
        return modes.get(mode_name, cls.STANDARD)
```

### 2.2 模式选择和执行

```python
def playmore_with_mode(keywords, mode="standard"):
    """
    根据模式处理论文
    
    mode: "fast" / "standard" / "deep"
    """
    
    # 获取模式配置
    mode_config = ProcessingMode.get_mode(mode)
    
    print(f"\n📋 {mode_config['description']}")
    print(f"   质量：{mode_config['quality']}")
    print(f"   Token估算：{mode_config['token_estimate']}")
    print(f"   耗时：{mode_config['time_estimate']}\n")
    
    # Step 1: fetch（所有模式都需要）
    print("🔍 搜索论文...")
    papers = playmore_fetch(keywords)
    print(f"   找到 {len(papers)} 篇论文")
    
    # Step 2: score（所有模式都需要）
    print("📊 打分排序...")
    scored_papers = playmore_score(papers)
    print(f"   保留 {len(scored_papers)} 篇高质量论文")
    
    # Step 3: review（标准和深度模式）
    if "review" in mode_config["steps"]:
        print("💬 AI点评...")
        reviewed_papers = playmore_review(scored_papers)
    else:
        reviewed_papers = scored_papers
    
    # Step 4: concepts（标准和深度模式）
    if "concepts" in mode_config["steps"]:
        print("🏷️  识别概念...")
        papers_with_concepts = extract_concepts_with_cache(reviewed_papers, mode)
    else:
        papers_with_concepts = reviewed_papers
    
    # Step 5: organize（仅深度模式）
    if "organize" in mode_config["steps"]:
        print("🔗 整理概念关系...")
        organized_concepts = organize_concepts(papers_with_concepts)
    else:
        organized_concepts = None
    
    # Step 6: notes（标准和深度模式）
    if "notes" in mode_config["steps"]:
        print("📝 生成笔记...")
        notes = playmore_notes(papers_with_concepts)
    else:
        notes = None
    
    # 输出结果
    print("\n✅ 处理完成！")
    print(f"   论文数：{len(papers_with_concepts)}")
    if organized_concepts:
        print(f"   概念数：{len(organized_concepts)}")
    if notes:
        print(f"   笔记数：{len(notes)}")
    
    return {
        "papers": papers_with_concepts,
        "concepts": organized_concepts,
        "notes": notes,
        "mode": mode
    }
```

### 2.3 命令行集成

```python
import argparse

def main():
    parser = argparse.ArgumentParser(description="Playmore - 学术论文追踪")
    
    parser.add_argument(
        "keywords",
        nargs="+",
        help="搜索关键词"
    )
    
    parser.add_argument(
        "--mode",
        choices=["fast", "standard", "deep"],
        default="standard",
        help="处理模式（默认：standard）"
    )
    
    parser.add_argument(
        "--fast",
        action="store_const",
        const="fast",
        dest="mode",
        help="快速模式（等同于 --mode fast）"
    )
    
    parser.add_argument(
        "--deep",
        action="store_const",
        const="deep",
        dest="mode",
        help="深度模式（等同于 --mode deep）"
    )
    
    parser.add_argument(
        "--no-cache",
        action="store_true",
        help="不使用缓存"
    )
    
    parser.add_argument(
        "--clear-cache",
        action="store_true",
        help="清空缓存"
    )
    
    parser.add_argument(
        "--cache-stats",
        action="store_true",
        help="显示缓存统计"
    )
    
    args = parser.parse_args()
    
    # 处理缓存命令
    cache = ConceptCache()
    
    if args.clear_cache:
        cache.cache = {
            "version": "1.0",
            "last_updated": datetime.now().isoformat(),
            "concepts_cache": {},
            "paper_cache": {},
            "search_cache": {}
        }
        cache._save_cache()
        print("✓ 缓存已清空")
        return
    
    if args.cache_stats:
        stats = cache.get_stats()
        print("\n📊 缓存统计：")
        print(f"   概念数：{stats['concepts_count']}")
        print(f"   论文数：{stats['papers_count']}")
        print(f"   搜索数：{stats['searches_count']}")
        print(f"   总命中：{stats['total_hits']}")
        print(f"   最后更新：{stats['last_updated']}")
        return
    
    # 处理搜索
    keywords = " ".join(args.keywords)
    result = playmore_with_mode(keywords, mode=args.mode)
```

---

## Phase 3：监控和优化

### 3.1 性能监控

```python
class PerformanceMonitor:
    """性能监控"""
    
    def __init__(self, log_file="~/Desktop/playmore/performance.log"):
        self.log_file = Path(log_file).expanduser()
        self.log_file.parent.mkdir(parents=True, exist_ok=True)
    
    def log_search(self, keywords, mode, token_used, time_used, cache_hit_rate):
        """记录搜索"""
        log_entry = {
            "timestamp": datetime.now().isoformat(),
            "keywords": keywords,
            "mode": mode,
            "token_used": token_used,
            "time_used": time_used,
            "cache_hit_rate": cache_hit_rate
        }
        
        with open(self.log_file, 'a') as f:
            f.write(json.dumps(log_entry) + "\n")
    
    def get_stats(self, days=7):
        """获取统计"""
        if not self.log_file.exists():
            return None
        
        entries = []
        with open(self.log_file, 'r') as f:
            for line in f:
                entry = json.loads(line)
                timestamp = datetime.fromisoformat(entry["timestamp"])
                if datetime.now() - timestamp < timedelta(days=days):
                    entries.append(entry)
        
        if not entries:
            return None
        
        return {
            "total_searches": len(entries),
            "avg_token_used": sum(e["token_used"] for e in entries) / len(entries),
            "avg_time_used": sum(e["time_used"] for e in entries) / len(entries),
            "avg_cache_hit_rate": sum(e["cache_hit_rate"] for e in entries) / len(entries),
            "mode_distribution": self._get_mode_distribution(entries)
        }
    
    @staticmethod
    def _get_mode_distribution(entries):
        """获取模式分布"""
        distribution = {}
        for entry in entries:
            mode = entry["mode"]
            distribution[mode] = distribution.get(mode, 0) + 1
        return distribution
```

### 3.2 缓存优化

```python
class CacheOptimizer:
    """缓存优化"""
    
    def __init__(self, cache):
        self.cache = cache
    
    def analyze_hit_rate(self):
        """分析命中率"""
        concepts = self.cache.cache["concepts_cache"]
        papers = self.cache.cache["paper_cache"]
        
        total_hits = sum(c.get("hit_count", 0) for c in concepts.values())
        total_hits += sum(p.get("hit_count", 0) for p in papers.values())
        
        total_items = len(concepts) + len(papers)
        
        if total_items == 0:
            return 0
        
        return total_hits / total_items
    
    def get_hot_concepts(self, top_n=10):
        """获取热门概念"""
        concepts = self.cache.cache["concepts_cache"]
        sorted_concepts = sorted(
            concepts.items(),
            key=lambda x: x[1].get("hit_count", 0),
            reverse=True
        )
        return sorted_concepts[:top_n]
    
    def get_cold_concepts(self, top_n=10):
        """获取冷门概念"""
        concepts = self.cache.cache["concepts_cache"]
        sorted_concepts = sorted(
            concepts.items(),
            key=lambda x: x[1].get("hit_count", 0)
        )
        return sorted_concepts[:top_n]
    
    def recommend_cleanup(self):
        """推荐清理"""
        cold_concepts = self.get_cold_concepts(top_n=20)
        
        recommendations = []
        for canonical_id, concept in cold_concepts:
            if concept.get("hit_count", 0) == 0:
                recommendations.append({
                    "type": "concept",
                    "id": canonical_id,
                    "name": concept["name"],
                    "reason": "从未被使用"
                })
        
        return recommendations
```

---

## 集成到现有系统

### 修改playmore-review的SKILL.md

```markdown
---
name: playmore-review
description: AI review and classification of scored papers. Classifies each as ⭐必读 / 📖可读 / 💡了解. Supports --fast/--standard/--deep modes with caching.
argument-hint: "[search keywords] [--mode fast|standard|deep]"
---

# Playmore Review

## 处理模式

### 快速模式（--fast）
- 只做基础打分
- Token消耗：3000-4000
- 质量：⭐⭐⭐
- 耗时：30秒

### 标准模式（--standard，默认）
- 完整分析 + 概念识别
- Token消耗：9640-13000
- 质量：⭐⭐⭐⭐⭐
- 耗时：2-3分钟

### 深度模式（--deep）
- 最全面分析 + 概念整理
- Token消耗：12000-16000
- 质量：⭐⭐⭐⭐⭐⭐
- 耗时：3-5分钟

## 缓存系统

- 自动缓存已识别的概念
- 缓存命中率：50-70%
- 长期节省成本：35-45%

## 命令示例

```bash
# 标准模式（推荐）
/playmore AI 心理健康

# 快速模式
/playmore --fast AI 心理健康

# 深度模式
/playmore --deep AI 心理健康

# 查看缓存统计
/playmore --cache-stats

# 清空缓存
/playmore --clear-cache
```
```

---

## 成本对比总结

### 实施前后

```
实施前（无优化）：
- 10篇论文：9640-13000 token
- 月度（40次）：385600-520000 token
- 年度成本：$48-72（Sonnet）

实施后（方案5）：
- 10篇论文：5000-8000 token（平均）
- 月度（40次）：250000-338000 token
- 年度成本：$30-48（Sonnet）

节省：
- 单次：35-45%
- 月度：135600-182000 token
- 年度：$18-24
```

---

## 实现检查清单

### Phase 1：缓存系统
- [ ] 设计缓存数据结构
- [ ] 实现ConceptCache类
- [ ] 实现缓存查询和更新
- [ ] 实现缓存过期清理
- [ ] 集成到概念提取流程
- [ ] 测试缓存命中率

### Phase 2：分层模式
- [ ] 定义三种处理模式
- [ ] 实现模式选择逻辑
- [ ] 实现命令行参数
- [ ] 实现模式特定的Prompt
- [ ] 测试不同模式的质量
- [ ] 编写用户文档

### Phase 3：监控优化
- [ ] 实现性能监控
- [ ] 实现缓存分析
- [ ] 实现优化建议
- [ ] 收集用户反馈
- [ ] 持续优化缓存策略

