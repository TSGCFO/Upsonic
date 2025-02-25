    
    @staticmethod
    def diagnose_missing_cache(cache_key):
        """Diagnose why a cache entry might be missing"""
        cache_key_full = f"cache_{cache_key}"
        
        # Check if the key exists in raw storage
        raw_data = ClientConfiguration.get(cache_key_full)
        
        if raw_data is None:
            return {
                'status': 'not_found',
                'reason': 'No data exists for this cache key',
                'suggestion': 'Verify the cache key is being generated consistently'
            }
            
        # Key exists but might not be retrievable via the cache API
        try:
            # Try to decode and deserialize
            decoded = base64.b64decode(raw_data)
            cache_data = cloudpickle.loads(decoded)
            
            # Check for expiration
            current_time = int(time.time())
            if current_time > cache_data.get('expiry_time', 0):
                return {
                    'status': 'expired',
                    'reason': 'Cache entry exists but has expired',
                    'expired_at': cache_data.get('expiry_time'),
                    'seconds_ago': current_time - cache_data.get('expiry_time'),
                    'suggestion': 'Cache is working correctly - entry was automatically expired'
                }
                
            # Data exists and isn't expired - should be retrievable
            return {
                'status': 'should_exist',
                'reason': 'Cache entry exists and is not expired',
                'expires_in': cache_data.get('expiry_time') - current_time,
                'suggestion': 'There may be an error in the cache retrieval code'
            }
            
        except Exception as e:
            return {
                'status': 'corrupt',
                'reason': f'Cache entry exists but is corrupt: {str(e)}',
                'suggestion': 'Delete this cache entry and recreate it'
            }
    
    @staticmethod
    def repair_cache():
        """Attempt to repair corrupt cache entries"""
        # Get all cache keys
        all_keys = ClientConfiguration.get_all_keys()
        cache_keys = [key for key in all_keys if key.startswith("cache_")]
        
        results = {
            'examined': len(cache_keys),
            'corrupt': 0,
            'expired': 0,
            'valid': 0,
            'deleted': 0,
            'details': []
        }
        
        for key in cache_keys:
            try:
                # Get the raw data
                raw_data = ClientConfiguration.get(key)
                if raw_data is None:
                    continue
                    
                # Try to decode and deserialize
                decoded = base64.b64decode(raw_data)
                cache_data = cloudpickle.loads(decoded)
                
                # Check for expiration
                current_time = int(time.time())
                if current_time > cache_data.get('expiry_time', 0):
                    # Expired entry
                    ClientConfiguration.delete(key)
                    results['expired'] += 1
                    results['deleted'] += 1
                    results['details'].append({
                        'key': key,
                        'status': 'expired',
                        'action': 'deleted'
                    })
                else:
                    # Valid entry
                    results['valid'] += 1
                    
            except Exception as e:
                # Corrupt entry
                ClientConfiguration.delete(key)
                results['corrupt'] += 1
                results['deleted'] += 1
                results['details'].append({
                    'key': key,
                    'status': 'corrupt',
                    'error': str(e),
                    'action': 'deleted'
                })
                
        return results
```

These diagnostic tools provide:
- **Serialization Testing**: Identify and isolate objects that can't be properly cached
- **Cache Inspection**: Determine why expected cache entries aren't being found
- **Automated Repair**: Clean up corrupted or expired cache entries
- **Detailed Reporting**: Get specific information about cache problems and solutions

### Cache Performance Analysis

Tools for understanding and optimizing cache efficiency:

```python
class CachePerformanceAnalyzer:
    """Tools for analyzing and optimizing cache performance"""
    
    def __init__(self):
        self.access_log = {}
        self.hits = 0
        self.misses = 0
        
    def track_access(self, cache_key, hit):
        """Record a cache access attempt"""
        if cache_key not in self.access_log:
            self.access_log[cache_key] = {
                'hits': 0,
                'misses': 0,
                'last_access': time.time()
            }
            
        entry = self.access_log[cache_key]
        entry['last_access'] = time.time()
        
        if hit:
            entry['hits'] += 1
            self.hits += 1
        else:
            entry['misses'] += 1
            self.misses += 1
    
    def get_with_tracking(self, cache_key):
        """Get from cache with performance tracking"""
        result = get_from_cache_with_expiry(cache_key)
        self.track_access(cache_key, result is not None)
        return result
        
    def set_with_tracking(self, data, cache_key, expiry_seconds):
        """Set cache with performance tracking"""
        save_to_cache_with_expiry(data, cache_key, expiry_seconds)
        # Track as a miss since we had to set it
        self.track_access(cache_key, False)
    
    def get_hit_rate(self):
        """Calculate overall cache hit rate"""
        total = self.hits + self.misses
        if total == 0:
            return 0
        return (self.hits / total) * 100
    
    def get_key_hit_rate(self, cache_key):
        """Calculate hit rate for a specific key"""
        if cache_key not in self.access_log:
            return None
            
        entry = self.access_log[cache_key]
        total = entry['hits'] + entry['misses']
        if total == 0:
            return 0
        return (entry['hits'] / total) * 100
    
    def get_performance_report(self):
        """Generate a comprehensive cache performance report"""
        # Calculate overall statistics
        total_accesses = self.hits + self.misses
        hit_rate = self.get_hit_rate()
        
        # Find most and least efficient cache keys
        key_stats = []
        for key, data in self.access_log.items():
            total = data['hits'] + data['misses']
            if total >= 5:  # Only consider keys with significant access
                key_stats.append({
                    'key': key,
                    'hit_rate': (data['hits'] / total) * 100 if total > 0 else 0,
                    'accesses': total,
                    'last_access': data['last_access']
                })
                
        # Sort by hit rate
        key_stats.sort(key=lambda x: x['hit_rate'], reverse=True)
        
        # Prepare report
        report = {
            'total_accesses': total_accesses,
            'hits': self.hits,
            'misses': self.misses,
            'overall_hit_rate': hit_rate,
            'unique_keys': len(self.access_log),
            'most_efficient': key_stats[:5] if key_stats else [],
            'least_efficient': key_stats[-5:] if len(key_stats) >= 5 else []
        }
        
        # Add optimization recommendations
        recommendations = []
        
        # Check overall hit rate
        if hit_rate < 50:
            recommendations.append(
                "Overall hit rate is low. Consider increasing cache duration."
            )
            
        # Look for keys with very low hit rates
        for key in report['least_efficient']:
            if key['hit_rate'] < 20 and key['accesses'] > 10:
                recommendations.append(
                    f"Key '{key['key']}' has a very low hit rate ({key['hit_rate']:.1f}%). "
                    f"Consider adjusting its caching strategy."
                )
                
        report['recommendations'] = recommendations
        return report
```

This analyzer provides:
- **Access Tracking**: Records all cache hits and misses
- **Performance Metrics**: Calculates hit rates and efficiency statistics
- **Key-Level Analysis**: Identifies which cache keys are performing well or poorly
- **Optimization Suggestions**: Provides specific recommendations for improvement
- **Usage Patterns**: Reveals how the cache is being used in practice

## Real-World Examples

### Content Management System Cache

Implementation for a CMS with smart caching:

```python
class CMSCache:
    """Content management system cache implementation"""
    
    def __init__(self, default_expiry=1800):
        self.default_expiry = default_expiry
        
    def get_page(self, page_id, version=None):
        """Get a CMS page with caching"""
        # Determine cache key based on page ID and optional version
        if version:
            cache_key = f"cms_page_{page_id}_v{version}"
        else:
            cache_key = f"cms_page_{page_id}"
            
        # Try to get from cache
        cached_page = get_from_cache_with_expiry(cache_key)
        if cached_page is not None:
            return {
                'source': 'cache',
                'page': cached_page
            }
            
        # Cache miss - fetch from database
        page = self._fetch_page_from_database(page_id, version)
        
        # Determine appropriate cache duration
        if page['status'] == 'published':
            # Published pages can be cached longer
            expiry = self.default_expiry
        elif page['status'] == 'draft':
            # Draft pages should expire quickly
            expiry = 300  # 5 minutes
        else:
            # Don't cache special status pages
            return {
                'source': 'database',
                'page': page
            }
            
        # Cache the page
        save_to_cache_with_expiry(
            data=page,
            cache_key=cache_key,
            expiry_seconds=expiry
        )
        
        return {
            'source': 'database',
            'page': page
        }
        
    def update_page(self, page_id, content):
        """Update a page and manage cache invalidation"""
        # Update in database
        new_version = self._update_page_in_database(page_id, content)
        
        # Invalidate all versions of this page in cache
        self._invalidate_page_cache(page_id)
        
        return {
            'success': True,
            'page_id': page_id,
            'new_version': new_version
        }
        
    def _invalidate_page_cache(self, page_id):
        """Invalidate all cached versions of a page"""
        # Get all cache keys
        all_keys = ClientConfiguration.get_all_keys()
        
        # Find and delete all keys for this page
        pattern = f"cache_cms_page_{page_id}"
        for key in all_keys:
            if key.startswith(pattern):
                ClientConfiguration.delete(key)
                
    def _fetch_page_from_database(self, page_id, version=None):
        """Simulate fetching a page from database"""
        # This would actually query a database in a real implementation
        return {
            'id': page_id,
            'title': f"Page {page_id}",
            'content': f"Content for page {page_id}",
            'version': version or 1,
            'status': 'published',
            'last_updated': time.time()
        }
        
    def _update_page_in_database(self, page_id, content):
        """Simulate updating a page in database"""
        # This would actually update a database in a real implementation
        return int(time.time())  # Return new version
```

This CMS cache implementation demonstrates:
- **Content-Aware Caching**: Different caching rules based on content status
- **Selective Invalidation**: Targeted cache clearing when content changes
- **Version Management**: Support for different content versions
- **Transparent Operation**: Cache mechanics are hidden from the rest of the application

### Data Processing Pipeline

Implementation for an ETL pipeline with intelligent caching:

```python
class DataPipelineCache:
    """Caching system for data processing pipelines"""
    
    def __init__(self):
        self.debug_mode = False
        
    def process_dataset(self, dataset_id, transforms=None, force_reprocess=False):
        """Process a dataset with caching for expensive steps"""
        # Generate a cache key based on dataset and transforms
        transform_hash = self._hash_transforms(transforms or [])
        cache_key = f"dataset_{dataset_id}_process_{transform_hash}"
        
        # Check cache unless forced to reprocess
        if not force_reprocess:
            cached_result = get_from_cache_with_expiry(cache_key)
            if cached_result is not None:
                if self.debug_mode:
                    print(f"Using cached result for dataset {dataset_id}")
                return cached_result
                
        # Cache miss or forced reprocess
        if self.debug_mode:
            print(f"Processing dataset {dataset_id} from scratch")
            
        # Actual data processing logic
        raw_data = self._load_dataset(dataset_id)
        
        # Apply each transform in sequence
        processed_data = raw_data
        for transform in (transforms or []):
            transform_start = time.time()
            processed_data = transform(processed_data)
            transform_time = time.time() - transform_start
            
            if self.debug_mode:
                print(f"Transform {transform.__name__} took {transform_time:.2f}s")
                
        # Cache the final result
        save_to_cache_with_expiry(
            data=processed_data,
            cache_key=cache_key,
            expiry_seconds=3600  # 1 hour default
        )
        
        return processed_data
        
    def invalidate_dataset_cache(self, dataset_id=None):
        """Invalidate cache for a dataset or all datasets"""
        all_keys = ClientConfiguration.get_all_keys()
        
        if dataset_id:
            # Invalidate specific dataset
            pattern = f"cache_dataset_{dataset_id}_"
            count = 0
            for key in all_keys:
                if key.startswith(pattern):
                    ClientConfiguration.delete(key)
                    count += 1
            return {
                'invalidated': count,
                'dataset_id': dataset_id
            }
        else:
            # Invalidate all datasets
            pattern = "cache_dataset_"
            count = 0
            for key in all_keys:
                if key.startswith(pattern):
                    ClientConfiguration.delete(key)
                    count += 1
            return {
                'invalidated': count,
                'all_datasets': True
            }
            
    def _hash_transforms(self, transforms):
        """Create a hash of the transform pipeline"""
        # Get names and source code of transforms when possible
        transform_info = []
        for transform in transforms:
            if hasattr(transform, '__name__'):
                name = transform.__name__
            else:
                name = str(transform)
                
            if hasattr(transform, '__code__'):
                # Include actual function code in the hash
                code_hash = hashlib.md5(str(transform.__code__.co_code).encode()).hexdigest()
                transform_info.append(f"{name}:{code_hash}")
            else:
                transform_info.append(name)
                
        # Create a hash of the entire pipeline
        pipeline_str = "|".join(transform_info)
        return hashlib.md5(pipeline_str.encode()).hexdigest()
        
    def _load_dataset(self, dataset_id):
        """Simulate loading a dataset"""
        # This would actually load data in a real implementation
        return {
            'id': dataset_id,
            'rows': 1000,
            'data': [i for i in range(1000)]
        }
```

This data pipeline cache demonstrates:
- **Transform Awareness**: Cache keys include the actual transformations being applied
- **Selective Reprocessing**: Option to force reprocessing when needed
- **Code-Based Hashing**: Cache keys account for the actual code in transform functions
- **Debug Support**: Optional detailed logging for cache operations
- **Targeted Invalidation**: Can clear cache for specific datasets or all datasets

## Conclusion

The Upsonic caching system provides a powerful yet easy-to-use mechanism for improving application performance while maintaining data freshness. By thoughtfully implementing caching in your Upsonic projects, you can achieve:

1. **Dramatic Performance Improvements**: Eliminate redundant expensive operations
2. **Reduced External Service Load**: Minimize API calls and database queries
3. **Improved User Experience**: Deliver responses faster through cached results
4. **Resource Optimization**: Lower processing and bandwidth requirements
5. **Intelligent Data Management**: Keep data fresh while maximizing cache effectiveness

The intelligent design of the caching system - with features like automatic serialization, time-based expiration, and error resilience - means you can focus on your application logic while the caching layer transparently handles the complexities of data persistence and retrieval.### Cache Invalidation Management

For controlled cache invalidation beyond time-based expiry:

```python
class CacheManager:
    """Advanced cache management system"""
    
    @staticmethod
    def invalidate_by_prefix(prefix, confirm=False):
        """Invalidate all cache entries with matching prefix"""
        if not confirm:
            return {
                'status': 'error',
                'message': 'Invalidation requires confirmation. Set confirm=True.'
            }
            
        # Get all cache keys from configuration
        all_keys = ClientConfiguration.get_all_keys()
        
        # Filter for cache keys with the given prefix
        matching_keys = [
            key for key in all_keys 
            if key.startswith(f"cache_{prefix}")
        ]
        
        # Delete all matching keys
        invalidated = []
        for key in matching_keys:
            ClientConfiguration.delete(key)
            # Strip the 'cache_' prefix for reporting
            invalidated.append(key[6:])
            
        return {
            'status': 'success',
            'invalidated_count': len(invalidated),
            'invalidated_keys': invalidated
        }
    
    @staticmethod
    def set_new_expiry(cache_key, new_expiry_seconds):
        """Update the expiration time for an existing cache entry"""
        # Get the existing cache entry
        cache_key_full = f"cache_{cache_key}"
        serialized_data = ClientConfiguration.get(cache_key_full)
        
        if serialized_data is None:
            return {
                'status': 'error',
                'message': 'Cache entry not found'
            }
            
        try:
            # Deserialize the cache entry
            cache_data = cloudpickle.loads(base64.b64decode(serialized_data))
            
            # Update the expiration time
            current_time = int(time.time())
            cache_data['expiry_time'] = current_time + new_expiry_seconds
            
            # Serialize and save the updated entry
            updated_data = base64.b64encode(cloudpickle.dumps(cache_data)).decode('utf-8')
            ClientConfiguration.set(cache_key_full, updated_data)
            
            return {
                'status': 'success',
                'new_expiry_time': cache_data['expiry_time'],
                'seconds_until_expiry': new_expiry_seconds
            }
        except Exception as e:
            # If anything goes wrong, invalidate the cache entry
            ClientConfiguration.delete(cache_key_full)
            return {
                'status': 'error',
                'message': f'Error updating expiry: {str(e)}',
                'cache_invalidated': True
            }
    
    @staticmethod
    def get_cache_info(cache_key):
        """Get metadata about a cache entry without retrieving the full data"""
        cache_key_full = f"cache_{cache_key}"
        serialized_data = ClientConfiguration.get(cache_key_full)
        
        if serialized_data is None:
            return {
                'status': 'not_found',
                'exists': False
            }
            
        try:
            # Deserialize the cache entry
            cache_data = cloudpickle.loads(base64.b64decode(serialized_data))
            
            # Calculate remaining time
            current_time = int(time.time())
            remaining_seconds = max(0, cache_data['expiry_time'] - current_time)
            
            # Determine cache status
            if current_time > cache_data['expiry_time']:
                status = 'expired'
            else:
                status = 'valid'
                
            return {
                'status': status,
                'exists': True,
                'created_at': cache_data['created_at'],
                'expiry_time': cache_data['expiry_time'],
                'remaining_seconds': remaining_seconds,
                'data_type': type(cache_data['data']).__name__
            }
        except Exception:
            return {
                'status': 'corrupt',
                'exists': True,
                'readable': False
            }
```

This advanced management system provides:
- **Pattern-Based Invalidation**: Clear multiple related cache entries at once
- **Expiry Adjustment**: Extend or reduce cache lifetime based on changing needs
- **Metadata Inspection**: Examine cache properties without loading full data
- **Corruption Detection**: Identify and report problematic cache entries
- **Safe Operations**: Confirmation requirements for destructive operations

### Tiered Caching Strategy

For optimizing cache performance based on access patterns:

```python
class TieredCache:
    """Multi-level caching system with different expiration policies"""
    
    def __init__(self):
        # Define tiered cache settings
        self.tiers = {
            'frequent': {
                'prefix': 'tier1_',
                'expiry': 300,  # 5 minutes for frequently accessed data
                'access_threshold': 5
            },
            'standard': {
                'prefix': 'tier2_',
                'expiry': 3600,  # 1 hour for standard data
                'access_threshold': 0
            },
            'archival': {
                'prefix': 'tier3_',
                'expiry': 86400,  # 24 hours for rarely accessed data
                'promotion_threshold': None  # Cannot be promoted
            }
        }
        
        # Access tracking
        self._access_counts = {}
        
    def _get_tier_key(self, tier, base_key):
        """Generate a tier-specific cache key"""
        tier_config = self.tiers.get(tier)
        if not tier_config:
            raise ValueError(f"Unknown tier: {tier}")
        return f"{tier_config['prefix']}{base_key}"
        
    def get(self, key, default_tier='standard'):
        """Get data from the appropriate cache tier"""
        # Try each tier in order of speed/recency
        for tier_name in ['frequent', 'standard', 'archival']:
            tier_key = self._get_tier_key(tier_name, key)
            result = get_from_cache_with_expiry(tier_key)
            
            if result is not None:
                # Track access for potential tier promotion
                self._track_access(key, tier_name)
                
                # Consider promoting to a faster tier if accessed frequently
                self._consider_promotion(key, tier_name)
                
                return result
                
        # Cache miss across all tiers
        return None
        
    def set(self, key, data, tier='standard'):
        """Store data in the specified cache tier"""
        tier_config = self.tiers.get(tier)
        if not tier_config:
            raise ValueError(f"Unknown tier: {tier}")
            
        tier_key = self._get_tier_key(tier, key)
        save_to_cache_with_expiry(
            data=data,
            cache_key=tier_key,
            expiry_seconds=tier_config['expiry']
        )
        
        # Initialize access tracking
        self._access_counts[key] = self._access_counts.get(key, 0)
        
        return True
        
    def _track_access(self, key, tier):
        """Track number of times a key is accessed"""
        if key not in self._access_counts:
            self._access_counts[key] = 0
            
        self._access_counts[key] += 1
        
    def _consider_promotion(self, key, current_tier):
        """Consider promoting a frequently accessed key to a faster tier"""
        access_count = self._access_counts.get(key, 0)
        
        # Check promotion criteria
        if current_tier == 'standard' and access_count >= self.tiers['frequent']['access_threshold']:
            # Promote standard → frequent
            standard_key = self._get_tier_key('standard', key)
            data = get_from_cache_with_expiry(standard_key)
            
            if data is not None:
                frequent_key = self._get_tier_key('frequent', key)
                save_to_cache_with_expiry(
                    data=data,
                    cache_key=frequent_key,
                    expiry_seconds=self.tiers['frequent']['expiry']
                )
                
        elif current_tier == 'archival' and self.tiers['standard']['access_threshold'] is not None:
            # Promote archival → standard (if configured)
            if access_count >= self.tiers['standard']['access_threshold']:
                archival_key = self._get_tier_key('archival', key)
                data = get_from_cache_with_expiry(archival_key)
                
                if data is not None:
                    standard_key = self._get_tier_key('standard', key)
                    save_to_cache_with_expiry(
                        data=data,
                        cache_key=standard_key,
                        expiry_seconds=self.tiers['standard']['expiry']
                    )
```

This tiered caching system implements:
- **Access-Based Optimization**: Frequently accessed data moves to shorter-lived but faster-access tiers
- **Automatic Promotion**: Data migrates between tiers based on usage patterns
- **Differentiated Expiration**: Different tiers have appropriate expiration policies
- **Transparent Lookup**: Clients can access data without knowing which tier contains it
- **Hierarchical Storage**: Mimics memory hierarchy principles from computer architecture

## Implementation Details

### Serialization Approach

The caching system uses a sophisticated serialization strategy to handle complex Python objects:

```python
def serialize_cache_data(data):
    """Demonstrate the serialization approach used in the caching system"""
    # First, detect the module of the data to ensure proper pickling
    the_module = dill.detect.getmodule(data)
    
    # Register the module with cloudpickle if found
    if the_module is not None:
        cloudpickle.register_pickle_by_value(the_module)
    
    # Create the cache entry structure with metadata
    cache_entry = {
        'data': data,
        'created_at': int(time.time()),
        'expiry_time': int(time.time()) + 3600,  # Example: 1 hour expiry
        'version': '1.0'  # Optional versioning
    }
    
    # Pickle the entire cache entry
    pickled_data = cloudpickle.dumps(cache_entry)
    
    # Encode to base64 for safe storage as string
    encoded_data = base64.b64encode(pickled_data).decode('utf-8')
    
    return encoded_data
```

The serialization process:

1. **Module Detection**: Uses `dill.detect.getmodule()` to identify the module where the data is defined, enabling serialization of classes and functions.

2. **Module Registration**: Registers the module with `cloudpickle.register_pickle_by_value()`, which tells cloudpickle to include the module's code in the pickle data rather than just referencing it.

3. **Structured Storage**: Creates a structured cache entry with metadata including creation time and expiration time.

4. **Cloudpickle Serialization**: Uses cloudpickle to convert the Python objects to a binary format, handling complex types like:
   - Custom classes with methods
   - Nested data structures
   - Function objects
   - Lambda functions
   - Objects with circular references
   - Module imports

5. **Base64 Encoding**: Converts the binary pickle data to a base64-encoded string for safe storage in text-based systems.

This approach ensures that virtually any Python object can be cached, regardless of complexity, while maintaining all its behavior when retrieved.

### Expiration Mechanism

The time-based expiration system operates as follows:

```python
def demonstrate_expiration_check(cache_key):
    """Demonstrate how expiration checking works"""
    cache_key_full = f"cache_{cache_key}"
    serialized_data = ClientConfiguration.get(cache_key_full)
    
    if serialized_data is None:
        return "Cache miss: No data found"
    
    try:
        # Deserialize the cache entry
        cache_data = cloudpickle.loads(base64.b64decode(serialized_data))
        
        # Get current time for expiration check
        current_time = int(time.time())
        
        # Check if the cache has expired
        if current_time > cache_data['expiry_time']:
            # Cache has expired, delete it
            ClientConfiguration.delete(cache_key_full)
            return "Cache expired: Entry deleted"
        
        # Calculate and display remaining time
        remaining_seconds = cache_data['expiry_time'] - current_time
        return f"Cache valid: {remaining_seconds} seconds remaining"
        
    except Exception as e:
        # Handle deserialization errors
        ClientConfiguration.delete(cache_key_full)
        return f"Cache error: {str(e)}, entry deleted"
```

The expiration system provides:

1. **Just-In-Time Validation**: Cache entries are checked for expiration only when accessed, not through a separate cleanup process.

2. **Self-Cleaning**: Expired entries are automatically deleted when detected, freeing up storage space.

3. **Unix Timestamp Comparison**: Simple integer comparison of current time vs. expiration time for efficient validation.

4. **Immediate Invalidation**: Expired data is never returned to the caller, maintaining data freshness guarantees.

5. **Error Handling**: Any deserialization issues trigger immediate cache invalidation, preventing corrupt data from persisting.

## Performance Considerations

### Optimizing Cache Key Design

Proper cache key design is critical for effective caching:

```python
class CacheKeyGenerator:
    """Best practices for creating effective cache keys"""
    
    @staticmethod
    def for_api_request(method, url, params=None, headers=None):
        """Generate cache key for API requests"""
        # Include only cache-relevant headers
        cache_headers = {}
        if headers:
            relevant_headers = ['accept', 'accept-language', 'content-type']
            cache_headers = {
                k.lower(): v for k, v in headers.items() 
                if k.lower() in relevant_headers
            }
        
        # Create a deterministic representation of the request
        components = [
            method.upper(),
            url.lower(),
            json.dumps(params or {}, sort_keys=True),
            json.dumps(cache_headers, sort_keys=True)
        ]
        
        # Generate a hash of the combined components
        key_material = '|'.join(components).encode()
        return f"api_{hashlib.md5(key_material).hexdigest()}"
    
    @staticmethod
    def for_computation(function_name, args, kwargs=None):
        """Generate cache key for computation results"""
        # Stringify the arguments
        arg_str = ','.join(str(arg) for arg in args)
        
        # Stringify the keyword arguments in a deterministic order
        kwargs_list = []
        if kwargs:
            for key in sorted(kwargs.keys()):
                kwargs_list.append(f"{key}={kwargs[key]}")
        kwargs_str = ','.join(kwargs_list)
        
        # Create the base key components
        components = [function_name, arg_str]
        if kwargs_str:
            components.append(kwargs_str)
            
        # Join and hash for a compact key
        key_material = '|'.join(components).encode()
        return f"compute_{hashlib.md5(key_material).hexdigest()}"
    
    @staticmethod
    def with_version(base_key, version):
        """Add versioning to cache keys for compatibility control"""
        return f"{base_key}_v{version}"
```

Effective cache key strategies include:

1. **Deterministic Generation**: Keys must be consistently reproducible for the same input parameters.

2. **Relevance Filtering**: Only cache-relevant aspects should be included in key generation.

3. **Namespacing**: Key prefixes help organize and manage different types of cached data.

4. **Versioning**: Version components allow for controlled cache invalidation when data formats change.

5. **Hashing**: Cryptographic hashing creates compact, fixed-length keys from variable-length inputs.

6. **Normalization**: Input normalization (lowercase, sorting) ensures consistent keys despite minor variations.

### Memory and Storage Efficiency

Optimizing caching resource usage:

```python
class CacheEfficiencyTools:
    """Tools for optimizing cache resource usage"""
    
    @staticmethod
    def estimate_size(data):
        """Estimate the storage size of cached data"""
        # Pickle the data to get realistic size estimate
        pickled = cloudpickle.dumps(data)
        base64_size = len(base64.b64encode(pickled))
        
        # Return sizes in bytes, KB
        return {
            'pickle_bytes': len(pickled),
            'base64_bytes': base64_size,
            'storage_kb': base64_size / 1024
        }
    
    @staticmethod
    def optimize_for_caching(data):
        """Optimize data structures for more efficient caching"""
        if isinstance(data, list):
            # Convert lists to tuples for immutability
            return tuple(
                CacheEfficiencyTools.optimize_for_caching(item) 
                for item in data
            )
        elif isinstance(data, dict):
            # Optimize dictionary values
            return {
                k: CacheEfficiencyTools.optimize_for_caching(v)
                for k, v in data.items()
            }
        elif hasattr(data, '__dict__') and not isinstance(data, type):
            # For custom objects, only pickle essential attributes
            if hasattr(data, 'cache_attributes'):
                # If object defines which attributes to cache
                return {
                    attr: CacheEfficiencyTools.optimize_for_caching(
                        getattr(data, attr)
                    )
                    for attr in data.cache_attributes
                    if hasattr(data, attr)
                }
            else:
                # Default behavior
                return data
        else:
            # No optimization needed for simple types
            return data
    
    @staticmethod
    def monitor_cache_usage(cache_prefix=''):
        """Monitor current cache resource usage"""
        # Get all cache keys from configuration
        all_keys = ClientConfiguration.get_all_keys()
        
        # Filter for cache keys with the given prefix
        if cache_prefix:
            cache_keys = [
                key for key in all_keys 
                if key.startswith(f"cache_{cache_prefix}")
            ]
        else:
            cache_keys = [
                key for key in all_keys 
                if key.startswith("cache_")
            ]
        
        # Gather statistics
        total_entries = len(cache_keys)
        total_size = 0
        entry_sizes = {}
        
        for key in cache_keys:
            data = ClientConfiguration.get(key)
            if data:
                size = len(data)
                total_size += size
                entry_sizes[key] = size
        
        # Find largest entries
        sorted_entries = sorted(
            entry_sizes.items(), 
            key=lambda x: x[1],
            reverse=True
        )
        largest_entries = sorted_entries[:5] if sorted_entries else []
        
        return {
            'total_entries': total_entries,
            'total_size_bytes': total_size,
            'total_size_kb': total_size / 1024,
            'average_size_bytes': total_size / total_entries if total_entries else 0,
            'largest_entries': largest_entries
        }
```

These tools enable:

1. **Size Estimation**: Understanding the storage requirements for different cached data structures.

2. **Data Optimization**: Transforming data to more cache-efficient forms before storage.

3. **Usage Monitoring**: Tracking overall cache resource consumption and identifying problematic entries.

4. **Selective Caching**: Making informed decisions about what data should be cached based on size considerations.

5. **Resource Management**: Identifying opportunities for cache cleanup or optimization.

## Best Practices

### When to Use Caching

Guidelines for effective cache usage:

```python
def should_cache(operation_type, data_size_kb, computation_time_ms, access_frequency):
    """Determine if an operation should use caching"""
    
    # Define thresholds for different operation types
    thresholds = {
        'api_call': {
            'min_time_ms': 100,  # API calls over 100ms should be cached
            'min_frequency': 0.1  # Even infrequent API calls benefit from caching
        },
        'computation': {
            'min_time_ms': 500,  # Computations over 500ms should be cached
            'min_frequency': 0.5  # Higher frequency threshold for computations
        },
        'database_query': {
            'min_time_ms': 200,  # Database queries over 200ms
            'min_frequency': 0.3  # Moderate frequency threshold
        }
    }
    
    # Get thresholds for this operation type
    type_thresholds = thresholds.get(operation_type, {
        'min_time_ms': 1000,    # Default: cache if over 1 second
        'min_frequency': 0.5     # Default: cache if accessed at least every other request
    })
    
    # Size considerations
    if data_size_kb > 5000:  # 5MB
        # For very large data, only cache if highly beneficial
        size_factor = 0.5  # Reduces likelihood for large data
    elif data_size_kb > 1000:  # 1MB
        size_factor = 0.7  # Slightly reduces likelihood
    else:
        size_factor = 1.0  # No reduction for normal-sized data
    
    # Calculate cache benefit score
    benefit_score = (
        (computation_time_ms / type_thresholds['min_time_ms']) *
        (access_frequency / type_thresholds['min_frequency']) *
        size_factor
    )
    
    # Provide decision with rationale
    should_use_cache = benefit_score >= 1.0
    
    return {
        'should_cache': should_use_cache,
        'benefit_score': benefit_score,
        'rationale': {
            'computation_ratio': computation_time_ms / type_thresholds['min_time_ms'],
            'frequency_ratio': access_frequency / type_thresholds['min_frequency'],
            'size_factor': size_factor
        },
        'recommendation': "Cache" if should_use_cache else "Don't cache",
        'suggested_expiry': recommend_expiry(operation_type, access_frequency)
    }

def recommend_expiry(operation_type, access_frequency):
    """Recommend appropriate cache expiry time based on operation type and access pattern"""
    
    # Base expiry times for different operation types (in seconds)
    base_expiry = {
        'api_call': 300,        # 5 minutes for API calls
        'computation': 1800,     # 30 minutes for computations
        'database_query': 600,   # 10 minutes for database queries
        'model_prediction': 3600 # 1 hour for model predictions
    }
    
    # Get base expiry for this operation type
    expiry = base_expiry.get(operation_type, 900)  # Default: 15 minutes
    
    # Adjust based on access frequency
    if access_frequency > 10:    # Accessed more than 10 times per hour
        expiry = expiry * 0.5    # Shorter expiry for frequently changing data
    elif access_frequency < 0.1: # Accessed less than once per 10 hours
        expiry = expiry * 2      # Longer expiry for rarely accessed data
    
    return int(expiry)
```

This decision framework incorporates:

1. **Operation-Specific Thresholds**: Different operations have different caching characteristics.

2. **Multi-Factor Analysis**: Considers computation time, access frequency, and data size.

3. **Size-Based Adjustments**: Larger data requires more significant benefits to justify caching.

4. **Expiry Recommendations**: Suggests appropriate cache lifetimes based on operation and access patterns.

5. **Transparent Decision Logic**: Provides clear rationale for caching decisions.

### Cache Validation Strategies

Ensuring cache correctness beyond expiration:

```python
class CacheValidator:
    """Strategies for validating cache freshness and correctness"""
    
    @staticmethod
    def with_etag(cache_key, fetch_function, current_etag=None):
        """Validate cache using ETag pattern for HTTP resources"""
        # Try to get cached response and metadata
        cached_data = get_from_cache_with_expiry(cache_key)
        
        if cached_data and isinstance(cached_data, dict) and 'etag' in cached_data:
            # Check if we have a new ETag to compare against
            if current_etag and cached_data['etag'] == current_etag:
                # ETag matches, cache is still valid
                return {
                    'source': 'cache',
                    'data': cached_data['data'],
                    'validation': 'etag_match'
                }
            
            # No ETag provided or different ETag - need to validate with source
            try:
                # Fetch with If-None-Match header if we have an ETag
                result = fetch_function(
                    if_none_match=cached_data.get('etag')
                )
                
                if result.get('status_code') == 304:  # Not Modified
                    # Resource hasn't changed, update expiry but keep data
                    save_to_cache_with_expiry(
                        data=cached_data,
                        cache_key=cache_key,
                        expiry_seconds=3600  # Reset expiry
                    )
                    return {
                        'source': 'cache',
                        'data': cached_data['data'],
                        'validation': 'server_validated'
                    }
                else:
                    # Resource has changed, update cache
                    new_data = {
                        'data': result.get('data'),
                        'etag': result.get('etag'),
                        'last_modified': result.get('last_modified')
                    }
                    save_to_cache_with_expiry(
                        data=new_data,
                        cache_key=cache_key,
                        expiry_seconds=3600
                    )
                    return {
                        'source': 'server',
                        'data': new_data['data'],
                        'validation': 'content_changed'
                    }
            except Exception as e:
                # On error, fall back to cached data if available
                if cached_data:
                    return {
                        'source': 'cache',
                        'data': cached_data['data'],
                        'validation': 'fallback_on_error',
                        'error': str(e)
                    }
                raise
        
        # No valid cache - fetch from source
        try:
            result = fetch_function()
            new_data = {
                'data': result.get('data'),
                'etag': result.get('etag'),
                'last_modified': result.get('last_modified')
            }
            save_to_cache_with_expiry(
                data=new_data,
                cache_key=cache_key,
                expiry_seconds=3600
            )
            return {
                'source': 'server',
                'data': new_data['data'],
                'validation': 'initial_fetch'
            }
        except Exception as e:
            # Still try to use stale cache in case of emergency
            if cached_data:
                return {
                    'source': 'stale_cache',
                    'data': cached_data['data'],
                    'validation': 'stale_fallback',
                    'error': str(e)
                }
            raise
    
    @staticmethod
    def with_version_check(cache_key, current_version, fetch_function):
        """Validate cache using version comparison"""
        # Try to get cached data
        cached_data = get_from_cache_with_expiry(cache_key)
        
        if cached_data and isinstance(cached_data, dict) and 'version' in cached_data:
            # Compare versions
            if cached_data['version'] == current_version:
                # Version matches, cache is valid
                return {
                    'source': 'cache',
                    'data': cached_data['data'],
                    'version': cached_data['version']
                }
        
        # No valid cache or version mismatch - fetch new data
        new_data = fetch_function()
        
        # Cache the new data with version
        cache_entry = {
            'data': new_data,
            'version': current_version
        }
        save_to_cache_with_expiry(
            data=cache_entry,
            cache_key=cache_key,
            expiry_seconds=3600
        )
        
        return {
            'source': 'fresh',
            'data': new_data,
            'version': current_version
        }
```

These validation strategies provide:

1. **ETag-Based Validation**: Uses HTTP ETags for efficient resource validation without transferring full content.

2. **Version-Based Validation**: Compares version identifiers to detect when cached data needs refreshing.

3. **Graceful Degradation**: Falls back to stale cache data when fresh data cannot be retrieved.

4. **Efficient Updates**: Avoids unnecessary data transfers when content hasn't changed.

5. **Metadata-Enhanced Caching**: Stores validation metadata alongside the actual cached content.

## Troubleshooting

### Common Caching Issues

Diagnosing and resolving cache-related problems:

```python
class CacheDiagnostics:
    """Tools for diagnosing and fixing common cache issues"""
    
    @staticmethod
    def diagnose_serialization_error(data):
        """Test if data can be properly serialized for caching"""
        try:
            # Try to pickle the data
            the_module = dill.detect.getmodule(data)
            if the_module is not None:
                cloudpickle.register_pickle_by_value(the_module)
                
            # Attempt full serialization process
            pickled = cloudpickle.dumps(data)
            encoded = base64.b64encode(pickled)
            
            # Verify roundtrip
            decoded = base64.b64decode(encoded)
            unpickled = cloudpickle.loads(decoded)
            
            # Check if data looks the same after roundtrip
            success = True
            if isinstance(data, dict) and isinstance(unpickled, dict):
                if set(data.keys()) != set(unpickled.keys()):
                    success = False
            
            return {
                'serializable': True,
                'size_bytes': len(pickled),
                'encoded_size': len(encoded),
                'roundtrip_successful': success
            }
        except Exception as e:
            # If we can identify specific problematic parts
            if isinstance(data, dict):
                # Test each key-value pair
                problem_keys = []
                for key, value in data.items():
                    try:
                        cloudpickle.dumps(value)
                    except:
                        problem_keys.append(key)
                
                if problem_keys:
                    return {
                        'serializable': False,
                        'error': str(e),
                        'problem_keys': problem_keys,
                        'suggestion': 'Remove or modify problematic keys'
                    }
            
            return {
                'serializable': False,
                'error': str(e),
                'error_type': type(e).__name__,
                'suggestion': 'Data contains elements that cannot be pickled'
            }
    # Comprehensive Guide to Upsonic Caching System

## Introduction and Purpose

The Upsonic Caching System is a sophisticated persistence layer that enables temporary storage of complex data structures with automatic expiration. This system dramatically improves performance by eliminating redundant expensive operations while ensuring data freshness through configurable time-based invalidation.

Unlike simple key-value stores, the Upsonic caching system supports:

- **Complex Data Structures**: Store any Python object, including nested data structures, custom classes, and even functions
- **Automatic Serialization**: Transparent conversion of Python objects to storable format and back
- **Time-Based Expiration**: Configure exactly how long data should remain valid
- **Self-Healing**: Automatic cleanup of expired or corrupted cache entries
- **Error Resilience**: Graceful handling of serialization errors without disrupting application flow

This caching system is particularly valuable for AI operations that involve expensive computations, API calls, or large data processing tasks that produce identical results when run multiple times within a certain timeframe.

## Core Functionality

### Saving Data to Cache

The `save_to_cache_with_expiry()` function stores arbitrary data with a specified lifetime:

```python
from upsonic.storage.caching import save_to_cache_with_expiry

# Create complex data to cache
analysis_result = {
    "sentiment_scores": [0.8, 0.2, -0.3, 0.6],
    "key_entities": ["product", "customer", "service"],
    "summary": "Overall positive sentiment with concerns about service",
    "confidence": 0.92,
    "processing_time_ms": 2354
}

# Cache the result for 1 hour (3600 seconds)
save_to_cache_with_expiry(
    data=analysis_result,
    cache_key="sentiment_analysis_batch_12345",
    expiry_seconds=3600
)
```

**How It Works in Detail:**

1. **Module Registration**: First, the function identifies the module of the data object using `dill.detect.getmodule()` and registers it with cloudpickle, enabling proper serialization of complex objects that reference their own modules.

2. **Timestamp Creation**: The current Unix timestamp is captured, and an expiration timestamp is calculated by adding the specified expiry duration.

3. **Cache Entry Formation**: A structured cache entry is created, containing:
   - The actual data to be cached
   - The precise expiration timestamp
   - A creation timestamp for tracking purposes

4. **Safe Storage Process**:
   - Any existing cache with the same key is deleted to prevent partial updates
   - The cache entry is serialized using cloudpickle, which handles complex Python objects
   - The binary pickle data is base64-encoded to ensure safe storage as a string
   - The encoded data is stored in the ClientConfiguration system

5. **Error Protection**: The entire process is wrapped in a try-except block, ensuring that any failures (like serialization errors) result in cache deletion rather than corrupt entries.

### Retrieving Data from Cache

The `get_from_cache_with_expiry()` function retrieves cached data if it exists and is still valid:

```python
from upsonic.storage.caching import get_from_cache_with_expiry

# Try to get cached result
cached_analysis = get_from_cache_with_expiry("sentiment_analysis_batch_12345")

if cached_analysis is not None:
    # Cache hit - use the cached data
    print(f"Using cached result with confidence: {cached_analysis['confidence']}")
    process_result(cached_analysis)
else:
    # Cache miss - need to perform the operation again
    print("No valid cache found, performing analysis...")
    new_analysis = perform_sentiment_analysis()
    
    # Cache the new result for future use
    save_to_cache_with_expiry(
        data=new_analysis,
        cache_key="sentiment_analysis_batch_12345",
        expiry_seconds=3600
    )
    
    process_result(new_analysis)
```

**How It Works in Detail:**

1. **Key Formation**: The function builds the full cache key with the 'cache_' prefix for namespace isolation.

2. **Data Retrieval**: The serialized data is retrieved from the ClientConfiguration store.

3. **Null Handling**: If no data exists for this key, the function returns None immediately.

4. **Deserialization Process**:
   - The stored string is base64-decoded back to binary data
   - The binary data is deserialized using cloudpickle back into Python objects
   - This reconstruction preserves all the original object properties and relationships

5. **Expiration Verification**: The current timestamp is compared against the stored expiration time.
   - If the cache has expired, it is deleted and None is returned
   - If the cache is still valid, the original data is returned

6. **Error Resilience**: Any exceptions during retrieval or deserialization trigger automatic cache invalidation, preventing corrupt data from causing application errors.

## Practical Usage Patterns

### Expensive Computation Caching

This pattern optimizes performance for computationally intensive operations:

```python
from upsonic.storage.caching import get_from_cache_with_expiry, save_to_cache_with_expiry
import hashlib
import time

def compute_with_caching(input_data, force_refresh=False, cache_duration=1800):
    """Perform expensive computation with caching support"""
    # Generate a deterministic cache key from the input data
    input_hash = hashlib.md5(str(input_data).encode()).hexdigest()
    cache_key = f"expensive_computation_{input_hash}"
    
    # Try to get cached result if not forcing refresh
    if not force_refresh:
        cached_result = get_from_cache_with_expiry(cache_key)
        if cached_result is not None:
            print("Cache hit! Using cached computation result.")
            return cached_result
    
    # Cache miss or force refresh - perform the actual computation
    print("Cache miss. Performing expensive computation...")
    start_time = time.time()
    
    # Simulate expensive operation
    result = perform_expensive_computation(input_data)
    
    computation_time = time.time() - start_time
    print(f"Computation completed in {computation_time:.2f} seconds")
    
    # Cache the result for future use
    save_to_cache_with_expiry(
        data=result,
        cache_key=cache_key,
        expiry_seconds=cache_duration
    )
    
    return result
```

This pattern demonstrates several important caching best practices:
- **Deterministic Keys**: Creating cache keys based on input data hash ensures identical inputs use the same cache entry
- **Forced Refresh Option**: Provides a mechanism to bypass the cache when needed
- **Configurable Duration**: Allows different types of data to have appropriate lifetimes
- **Transparent Operation**: The caching layer is invisible to calling code, which simply gets results

### External API Response Caching

This pattern reduces API calls by caching responses:

```python
import requests
import json
from upsonic.storage.caching import get_from_cache_with_expiry, save_to_cache_with_expiry

class CachedAPIClient:
    """API client with built-in response caching"""
    
    def __init__(self, base_url, default_cache_seconds=300):
        self.base_url = base_url
        self.default_cache_seconds = default_cache_seconds
    
    def get(self, endpoint, params=None, cache_seconds=None, bypass_cache=False):
        """Make a GET request with caching support"""
        # Use default cache duration if not specified
        if cache_seconds is None:
            cache_seconds = self.default_cache_seconds
            
        # Build the cache key from the request details
        param_str = json.dumps(params or {}, sort_keys=True)
        cache_key = f"api_response_{self.base_url}_{endpoint}_{param_str}"
        
        # Try to get cached response
        if not bypass_cache:
            cached_response = get_from_cache_with_expiry(cache_key)
            if cached_response is not None:
                cached_response['from_cache'] = True
                return cached_response
                
        # Cache miss or bypass - make the actual API call
        url = f"{self.base_url}/{endpoint}"
        response = requests.get(url, params=params)
        
        # Process the response
        if response.status_code == 200:
            try:
                result = {
                    'status': 'success',
                    'data': response.json(),
                    'status_code': response.status_code,
                    'from_cache': False
                }
                
                # Cache successful responses
                save_to_cache_with_expiry(
                    data=result,
                    cache_key=cache_key,
                    expiry_seconds=cache_seconds
                )
                
                return result
            except ValueError:
                # Not JSON - return text
                return {
                    'status': 'success',
                    'text': response.text,
                    'status_code': response.status_code,
                    'from_cache': False
                }
        else:
            # Don't cache error responses
            return {
                'status': 'error',
                'error': response.text,
                'status_code': response.status_code,
                'from_cache': False
            }
```

This pattern showcases several advanced techniques:
- **Resource Conservation**: Prevents redundant API calls, saving bandwidth and reducing external service load
- **Response Transparency**: Indicates whether responses came from cache or live API
- **Selective Caching**: Only caches successful responses, not errors
- **Parameterized Keys**: Builds cache keys that account for all request parameters
- **Flexible Duration**: Allows different endpoints to have different cache lifetimes

### Model Response Caching

This pattern optimizes expensive AI model calls:

```python
from upsonic.storage.caching import get_from_cache_with_expiry, save_to_cache_with_expiry
import hashlib

class CachedModelPrediction:
    """AI model wrapper with prediction caching"""
    
    def __init__(self, model, cache_duration=3600):
        self.model = model
        self.cache_duration = cache_duration
        self.cache_hits = 0
        self.cache_misses = 0
        
    def predict(self, input_data, use_cache=True):
        """Make a prediction with optional caching"""
        # Generate a unique, deterministic cache key for this input
        input_hash = self._hash_input(input_data)
        cache_key = f"model_prediction_{self.model.name}_{input_hash}"
        
        # Try to get cached prediction
        if use_cache:
            cached_prediction = get_from_cache_with_expiry(cache_key)
            if cached_prediction is not None:
                self.cache_hits += 1
                return cached_prediction
                
        # Cache miss - run the actual prediction
        self.cache_misses += 1
        prediction = self.model.predict(input_data)
        
        # Cache the result
        save_to_cache_with_expiry(
            data=prediction,
            cache_key=cache_key,
            expiry_seconds=self.cache_duration
        )
        
        return prediction
        
    def _hash_input(self, input_data):
        """Create a consistent hash of the input data"""
        if isinstance(input_data, str):
            return hashlib.md5(input_data.encode()).hexdigest()
        elif isinstance(input_data, dict):
            # Sort dictionary to ensure consistent hashing
            serialized = json.dumps(input_data, sort_keys=True)
            return hashlib.md5(serialized.encode()).hexdigest()
        else:
            # For other types, convert to string first
            return hashlib.md5(str(input_data).encode()).hexdigest()
            
    def get_cache_stats(self):
        """Return cache hit/miss statistics"""
        total = self.cache_hits + self.cache_misses
        hit_rate = (self.cache_hits / total) * 100 if total > 0 else 0
        
        return {
            'hits': self.cache_hits,
            'misses': self.cache_misses,
            'total': total,
            'hit_rate': f"{hit_rate:.1f}%"
        }
```

This implementation demonstrates:
- **Resource Optimization**: Avoiding expensive model inference when results are cached
- **Statistics Tracking**: Monitoring cache effectiveness with hit/miss metrics
- **Input Normalization**: Ensuring consistent cache keys regardless of input format
- **Model Independence**: Working with any prediction model regardless of implementation

## Advanced Cache Management

### Cache Prefetching Strategy

For scenarios where you want to proactively cache data before it's needed:

```python
def prefetch_cache(expected_inputs, compute_function, cache_duration=1800):
    """Pre-compute and cache results for expected inputs"""
    results = {}
    
    for input_data in expected_inputs:
        # Generate cache key
        input_hash = hashlib.md5(str(input_data).encode()).hexdigest()
        cache_key = f"prefetched_{input_hash}"
        
        # Check if already cached
        existing = get_from_cache_with_expiry(cache_key)
        if existing is not None:
            results[input_data] = {
                'status': 'already_cached',
                'data': existing
            }
            continue
            
        # Compute and cache
        try:
            result = compute_function(input_data)
            save_to_cache_with_expiry(
                data=result,
                cache_key=cache_key,
                expiry_seconds=cache_duration
            )
            results[input_data] = {
                'status': 'cached',
                'data': result
            }
        except Exception as e:
            results[input_data] = {
                'status': 'error',
                'error': str(e)
            }
            
    return results
```

This strategy enables:
- **Reduced Latency**: Results are ready immediately when requested
- **Background Processing**: Expensive computations can run during idle time
- **Batch Efficiency**: Multiple computations can be processed in optimal order
- **Predicted Usage**: System can prepare for expected user demands
- **Load Distribution**: Peak computational loads can be smoothed out across time