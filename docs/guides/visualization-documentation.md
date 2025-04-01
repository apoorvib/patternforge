# Candlestick Visualization Module Documentation

## Overview

The `visualization.py` module provides comprehensive visualization capabilities for candlestick chart data and technical analysis patterns. It offers both static (Matplotlib) and interactive (Plotly) visualization options with extensive customization, caching mechanisms, and performance optimizations for large datasets.

This module enables the visualization of:
- Candlestick price data
- Technical patterns and indicators
- Market regimes and volatility states
- Volume profiles and analysis
- Multi-timeframe analysis
- Pattern reliability metrics

## Dependencies

The module relies on several key dependencies:

```python
# Common modules
import os, sys, threading, time, json, math, warnings
from collections import defaultdict, Counter
from datetime import datetime, timedelta
import logging, pickle

# Data processing
import numpy as np
import pandas as pd
from scipy import stats

# Visualization
import plotly.graph_objects as go
import plotly.express as px
from plotly.subplots import make_subplots
import matplotlib.pyplot as plt
import matplotlib.dates as mdates

# Machine learning
from sklearn.preprocessing import StandardScaler, RobustScaler
from sklearn.decomposition import PCA
from sklearn.cluster import DBSCAN, KMeans
from sklearn.mixture import GaussianMixture
from sklearn.metrics import silhouette_score, calinski_harabasz_score
```

## Core Classes

### VisualizationConfig

Configuration class for all visualization settings including color schemes, themes, fonts, and interactive features. This class allows for consistent styling across all visualizations.

#### Attributes

- `color_scheme` (Dict[str, str]): Color scheme for different elements
- `theme` (str): Theme name for plotly charts
- `default_height` (int): Default chart height
- `default_width` (int): Default chart width
- `pattern_opacity` (float): Opacity for pattern overlays
- `show_grid` (bool): Whether to show grid lines
- `annotation_font_size` (int): Font size for annotations
- `fonts` (Dict): Font settings
- `layout` (Dict): Layout settings
- `grid_settings` (Dict): Grid appearance settings
- `annotation_settings` (Dict): Annotation appearance settings
- `interactive_settings` (Dict): Settings for interactive features

#### Methods

- `update_theme(theme_name)`: Update visualization theme
- `_apply_theme_settings(theme_name)`: Apply settings for specific theme
- `to_dict()`: Convert config to dictionary
- `from_dict(config_dict)`: Create config from dictionary

### VisualizationCache

Cache manager for visualization components and calculations to improve performance when creating multiple charts or dashboards.

#### Attributes

- `max_size` (int): Maximum number of items in cache
- `ttl` (int): Time to live for cache items in seconds
- `_cache` (Dict): Main cache storage
- `_metadata` (Dict): Cache item metadata
- `_lock` (threading.Lock): Thread lock for cache operations

#### Methods

- `get_figure(key)`: Get cached figure
- `cache_figure(key, figure)`: Cache a figure
- `_is_expired(key)`: Check if cache item has expired
- `_remove_expired()`: Remove all expired items from cache
- `_evict_lru_item()`: Remove least recently used item from cache
- `_remove_item(key)`: Remove item from cache
- `_estimate_figure_size(figure)`: Estimate memory size of figure
- `get_cache_stats()`: Get cache statistics
- `clear()`: Clear all items from cache

### VisualizationTheme

Theme manager for visualization customization.

#### Attributes

- `theme_name` (str): Name of the theme
- `color_scheme` (Dict[str, str]): Theme color scheme
- `font_settings` (Dict[str, Any]): Theme font settings
- `chart_settings` (Dict[str, Any]): Theme chart settings

#### Methods

- `_get_default_colors()`: Get default color scheme
- `_get_default_fonts()`: Get default font settings
- `_get_default_chart_settings()`: Get default chart settings

### BaseVisualizationSettings

Provides common utilities and base settings for creating consistent visualizations across the package.

#### Attributes

- `config` (VisualizationConfig): Visualization configuration
- `cache` (VisualizationCache): Visualization cache

#### Methods

- `apply_default_layout(fig)`: Apply default layout settings to figure
- `create_annotation(text, x, y, is_pattern=False)`: Create figure annotation with default settings
- `get_color_for_value(value, is_bullish=True)`: Get appropriate color for a value
- `create_hover_template(fields, include_date=True)`: Create hover template with specified fields
- `add_range_selector(fig)`: Add default range selector to figure
- `format_number(value, precision=2, prefix='', suffix='')`: Format number with default settings
- `create_subplot_layout(num_rows, row_heights=None, shared_xaxis=True)`: Create subplot layout configuration
- `style_axis(fig, title='', row=1, axis='y', log_scale=False)`: Apply consistent axis styling
- `add_patterns_overlay(fig, patterns, row=1)`: Add pattern markers overlay to figure
- `create_color_scale(start_color, end_color, n_colors=10)`: Create a continuous color scale
- `apply_interactive_features(fig)`: Apply interactive features to figure

### CandlestickVisualizer

The main visualization class that creates candlestick charts and provides methods for adding patterns, indicators, and other analysis tools.

#### Initialization

```python
def __init__(self, df: pd.DataFrame, config: Optional[VisualizationConfig] = None):
    """
    Initialize visualizer with financial data and configuration
    
    Args:
        df (pd.DataFrame): DataFrame with OHLCV data
        config (VisualizationConfig, optional): Visualization configuration
        
    Raises:
        ValueError: If required columns are missing or data types are invalid
    """
```

#### Key Methods

- `create_candlestick_chart(use_plotly=False, **kwargs)`: Create a candlestick chart using either Plotly or Matplotlib
- `apply_theme(theme)`: Apply theme to visualizer
- `create_pattern_cluster_chart()`: Create a visualization of pattern clusters
- `create_multi_timeframe_chart(weekly_df, monthly_df)`: Create a multi-timeframe analysis chart
- `visualize_market_regimes(regimes)`: Create visualization of market regimes
- `create_pattern_reliability_chart(lookback_window=100)`: Create visualization of pattern reliability metrics
- `detect_failed_patterns(lookback_period=50, failure_threshold=0.02)`: Identify failed pattern setups and their characteristics
- `analyze_failure_patterns(failed_patterns)`: Analyze characteristics of failed patterns
- `get_failure_zones(failed_patterns, min_failures=3)`: Identify price zones with frequent pattern failures
- `analyze_pattern_completion_probability(lookback_period=100, min_occurrences=5)`: Analyze completion probability for detected patterns
- `analyze_historical_performance(min_occurrences=20, lookback_days=252)`: Analyze historical performance of detected patterns
- `predict_pattern_breakout(lookback_period=100, confidence_threshold=0.7)`: Predict potential pattern breakouts/breakdowns based on historical behavior
- `create_interactive_dashboard()`: Create an interactive dashboard with pattern filtering and analysis

### MarketRegime

Represents a market regime with its characteristics.

#### Attributes

- `regime_type` (str): e.g., 'trending', 'ranging', 'transitioning'
- `volatility` (str): e.g., 'high', 'medium', 'low'
- `trend` (str): e.g., 'bullish', 'bearish', 'neutral'
- `volume` (str): e.g., 'increasing', 'decreasing', 'stable'
- `start_date` (pd.Timestamp): Start date of the regime
- `end_date` (pd.Timestamp): End date of the regime
- `confidence` (float): Confidence level in the regime classification

### MarketRegimeAnalyzer

Analyzes and visualizes different market regimes (trending, ranging, volatile periods) to provide context for trading decisions.

#### Initialization

```python
def __init__(self, df: pd.DataFrame, window_size: int = 20, min_regime_duration: int = 5):
    """
    Initialize MarketRegimeAnalyzer
    
    Args:
        df (pd.DataFrame): OHLCV data
        window_size (int): Default window for calculations
        min_regime_duration (int): Minimum regime duration
        
    Raises:
        ValueError: If required columns are missing in DataFrame
    """
```

#### Key Methods

- `analyze_market_regime(window_size=20, volatility_window=20, trend_window=50)`: Analyze and identify market regimes
- `predict_next_regime(current_regime, confidence_threshold=0.6)`: Predict the most likely next regime
- `analyze_regime_stability(current_regime, window_size=20)`: Analyze stability of current market regime
- `analyze_transition_drivers(current_regime, window_size=20)`: Analyze potential drivers of regime transitions
- `filter_patterns_by_regime(patterns, min_reliability=0.6)`: Filter patterns based on current market regime reliability

## Advanced Components

### IndicatorManager

Manages lazy loading and caching of technical indicators.

#### Key Methods

- `get_indicator(name, params=None)`: Get indicator values, computing only if necessary
- `precompute_indicators(indicators)`: Precompute multiple indicators
- `clear_cache()`: Clear indicator cache

### DataManager

Manages data chunking and memory optimization for large datasets.

#### Key Methods

- `get_data_chunk(chunk_id)`: Get a specific data chunk
- `get_data_range(start_date, end_date)`: Get data for specified date range
- `optimize_memory()`: Optimize memory usage by clearing unused chunks
- `get_memory_usage()`: Get memory usage statistics

### ParallelProcessor

Manages parallel processing for computationally intensive tasks.

#### Key Methods

- `process_in_parallel(data, operation, **kwargs)`: Process data in parallel
- `parallel_indicator_calculation(data, indicators)`: Calculate multiple indicators in parallel
- `parallel_pattern_detection(data, patterns)`: Detect multiple patterns in parallel
- `parallel_regime_analysis(data, analysis_types)`: Perform multiple regime analyses in parallel

### ParallelTaskManager

Manages parallel task execution with error handling and monitoring.

#### Key Methods

- `submit_task(task_id, operation, data, **kwargs)`: Submit task for parallel execution
- `execute_pending_tasks()`: Execute all pending tasks in parallel
- `get_task_result(task_id)`: Get result of completed task
- `get_task_error(task_id)`: Get error from failed task
- `get_task_status(task_id)`: Get current task status

### TaskMonitor

Monitors task execution progress and system resources.

#### Key Methods

- `track_task(task_id)`: Start tracking new task
- `update_task_status(task_id, status)`: Update task status
- `get_task_status(task_id)`: Get current task status
- `get_performance_summary()`: Get performance summary for all tasks

### MultipleTimeframeSynchronizer

Synchronizes data and visualizations across multiple timeframes.

#### Key Methods

- `create_synchronized_view(timeframes=None)`: Create synchronized multi-timeframe view
- `add_synchronized_indicators(indicator_name, params=None, timeframes=None)`: Add technical indicator synchronized across timeframes
- `add_synchronized_patterns(pattern_types, timeframes=None)`: Add pattern detection synchronized across timeframes
- `analyze_cross_timeframe_signals(indicator_name, params=None)`: Analyze signals across multiple timeframes
- `confirm_patterns_across_timeframes(pattern_type, min_confirmations=2)`: Confirm pattern signals across multiple timeframes

### DrawingTools

Interactive drawing tools for technical analysis.

#### Key Methods

- `add_trendline(points, extend=True, style=None)`: Add trendline to chart
- `add_horizontal_line(y_value, label='', style=None)`: Add horizontal line to chart
- `add_fibonacci_retracement(high_point, low_point)`: Add Fibonacci retracement levels
- `add_rectangle(x_range, y_range, style=None)`: Add rectangle to chart
- `add_text_annotation(text, position, style=None)`: Add text annotation to chart
- `clear_all_drawings()`: Clear all drawings from chart
- `save_drawings(filename)`: Save current drawings to file
- `load_drawings(filename)`: Load drawings from file

### ConfigurationManager

Manages visualization settings and configuration persistence.

#### Key Methods

- `export_configuration(filename=None)`: Export current configuration to file
- `import_configuration(filename=None)`: Import configuration from file
- `set_user_preference(key, value)`: Set user preference
- `get_user_preference(key, default=None)`: Get user preference
- `reset_to_defaults()`: Reset configuration to defaults
- `save_session(session_name)`: Save current configuration as named session
- `get_saved_sessions()`: Get list of saved sessions

## Example Usage

### Creating Basic Candlestick Charts

```python
from patternforge.candlestick.visualization import CandlestickVisualizer
import pandas as pd

# Load OHLCV data
df = pd.read_csv('your_data.csv', parse_dates=['Date'], index_col='Date')

# Create visualizer
visualizer = CandlestickVisualizer(df)

# Create interactive chart with Plotly
fig = visualizer.create_candlestick_chart(use_plotly=True, title='Bitcoin Price Chart')

# Create static chart with Matplotlib
static_fig = visualizer.create_candlestick_chart(use_plotly=False)
```

### Adding Technical Patterns

```python
# Create visualization with pattern overlay
patterns = {}

# Add detected patterns
from patternforge.candlestick.patterns import CandlestickPatterns
pattern_detector = CandlestickPatterns()
patterns['engulfing'] = pattern_detector.detect_engulfing(df)
patterns['doji'] = pattern_detector.detect_doji(df)

# Create chart with patterns
fig = visualizer.create_candlestick_chart(use_plotly=True)
fig = visualizer.add_patterns_overlay(fig, patterns)
```

### Visualizing Market Regimes

```python
# Create market regime analyzer
from patternforge.candlestick.visualization import MarketRegimeAnalyzer
regime_analyzer = MarketRegimeAnalyzer(df)

# Analyze market regimes
regimes = regime_analyzer.analyze_market_regime()

# Visualize regimes
regime_chart = visualizer.visualize_market_regimes(regimes)
```

### Multi-Timeframe Analysis

```python
# Import data for different timeframes
import pandas as pd

# Assuming you have daily, weekly and monthly data
daily_df = pd.read_csv('daily_data.csv', parse_dates=['Date'], index_col='Date')
weekly_df = pd.read_csv('weekly_data.csv', parse_dates=['Date'], index_col='Date')
monthly_df = pd.read_csv('monthly_data.csv', parse_dates=['Date'], index_col='Date')

# Create visualizer
visualizer = CandlestickVisualizer(daily_df)

# Create multi-timeframe chart
mtf_chart = visualizer.create_multi_timeframe_chart(weekly_df, monthly_df)
```

### Creating Pattern Reliability Dashboard

```python
# Create pattern reliability analysis
reliability_chart = visualizer.create_pattern_reliability_chart(lookback_window=100)
```

### Creating Interactive Dashboard

```python
# Create full interactive dashboard
dashboard = visualizer.create_interactive_dashboard()
```

### Customization

The module provides extensive customization options through the `VisualizationConfig` class:

```python
from patternforge.candlestick.visualization import VisualizationConfig, CandlestickVisualizer
import pandas as pd

# Load data
df = pd.read_csv('your_data.csv', parse_dates=['Date'], index_col='Date')

# Create custom configuration
config = VisualizationConfig(
    color_scheme={
        'bullish': '#00ff00',
        'bearish': '#ff0000',
        'neutral': '#0000ff',
        'complex': '#800080',
        'volume_up': '#00ff00',
        'volume_down': '#ff0000',
        'background': '#ffffff',
        'text': '#000000'
    },
    theme='plotly_dark',
    default_height=1000,
    default_width=1500,
    pattern_opacity=0.6,
    show_grid=True
)

# Create visualizer with custom config
visualizer = CandlestickVisualizer(df, config=config)
```

## Advanced Examples

### Analyzing Pattern Reliability

```python
# Analyze pattern reliability
from patternforge.candlestick.visualization import CandlestickVisualizer
import pandas as pd

# Load data
df = pd.read_csv('your_data.csv', parse_dates=['Date'], index_col='Date')
visualizer = CandlestickVisualizer(df)

# Analyze historical performance of patterns
performance = visualizer.analyze_historical_performance(
    min_occurrences=20,  # Minimum pattern occurrences for analysis
    lookback_days=252    # Look back 1 year
)

# Print performance statistics
print(performance)
```

### Detecting Failed Patterns

```python
# Detect and analyze failed patterns
from patternforge.candlestick.visualization import CandlestickVisualizer
import pandas as pd

# Load data
df = pd.read_csv('your_data.csv', parse_dates=['Date'], index_col='Date')
visualizer = CandlestickVisualizer(df)

# Detect failed patterns
failed_patterns = visualizer.detect_failed_patterns(
    lookback_period=50,
    failure_threshold=0.02
)

# Analyze characteristics of failed patterns
analysis = visualizer.analyze_failure_patterns(failed_patterns)

# Identify zones with frequent failures
failure_zones = visualizer.get_failure_zones(
    failed_patterns,
    min_failures=3
)
```

### Predicting Breakouts

```python
# Predict pattern breakouts
from patternforge.candlestick.visualization import CandlestickVisualizer
import pandas as pd

# Load data
df = pd.read_csv('your_data.csv', parse_dates=['Date'], index_col='Date')
visualizer = CandlestickVisualizer(df)

# Predict potential breakouts
breakout_predictions = visualizer.predict_pattern_breakout(
    lookback_period=100,
    confidence_threshold=0.7
)

# Filter for high-confidence breakouts
high_confidence = breakout_predictions[breakout_predictions['confidence'] > 0.8]
```

### Multi-Timeframe Confirmation

```python
# Confirm patterns across timeframes
from patternforge.candlestick.visualization import MultipleTimeframeSynchronizer
import pandas as pd

# Load data for different timeframes
daily = pd.read_csv('daily_data.csv', parse_dates=['Date'], index_col='Date')
weekly = pd.read_csv('weekly_data.csv', parse_dates=['Date'], index_col='Date')
monthly = pd.read_csv('monthly_data.csv', parse_dates=['Date'], index_col='Date')

# Create timeframe synchronizer
sync = MultipleTimeframeSynchronizer(daily)

# Add data for other timeframes
sync.timeframes['1W'] = weekly
sync.timeframes['1M'] = monthly

# Confirm patterns across timeframes
confirmed_patterns = sync.confirm_patterns_across_timeframes(
    pattern_type='engulfing',
    min_confirmations=2
)

# Create synchronized visualization
fig = sync.create_synchronized_view()

# Add synchronized patterns
fig = sync.add_synchronized_patterns(['engulfing', 'doji'])
```

## Performance Optimization

The module includes several features for optimizing performance with large datasets:

### Visualization Caching

```python
from patternforge.candlestick.visualization import CandlestickVisualizer, VisualizationCache
import pandas as pd

# Load data
df = pd.read_csv('your_data.csv', parse_dates=['Date'], index_col='Date')
visualizer = CandlestickVisualizer(df)

# Cache visualization for reuse
cache = VisualizationCache(max_size=100, ttl=3600)  # 1 hour cache lifetime
fig = visualizer.create_candlestick_chart(use_plotly=True)

# Cache the figure
cache.cache_figure('candlestick_main', fig)

# Later, retrieve from cache
cached_fig = cache.get_figure('candlestick_main')
```

### Data Chunking

```python
from patternforge.candlestick.visualization import DataManager
import pandas as pd

# Load large dataset
df = pd.read_csv('large_dataset.csv', parse_dates=['Date'], index_col='Date')

# Create data manager with chunking
data_manager = DataManager(df, chunk_size=1000, max_chunks=10)

# Get specific date range efficiently
start_date = pd.Timestamp('2022-01-01')
end_date = pd.Timestamp('2022-03-31')
data_chunk = data_manager.get_data_range(start_date, end_date)

# Optimize memory usage
data_manager.optimize_memory()
```

### Parallel Processing

```python
from patternforge.candlestick.visualization import ParallelProcessor
import pandas as pd

# Load large dataset
df = pd.read_csv('large_dataset.csv', parse_dates=['Date'], index_col='Date')

# Define operation to run in parallel
def calculate_moving_averages(chunk, window=20):
    return chunk['Close'].rolling(window=window).mean()

# Process in parallel
with ParallelProcessor(max_workers=4) as processor:
    results = processor.process_in_parallel(df, calculate_moving_averages, window=20)
```

## Troubleshooting

### Common Issues

1. **Missing Required Columns**

   The `CandlestickVisualizer` requires OHLC data columns. If you get a "Missing required columns" error, ensure your DataFrame has 'Open', 'High', 'Low', and 'Close' columns.

2. **Invalid Date Index**

   Many functions expect the DataFrame to have a DatetimeIndex. Convert your date column to a DatetimeIndex:

   ```python
   df['Date'] = pd.to_datetime(df['Date'])
   df.set_index('Date', inplace=True)
   ```

3. **Memory Issues with Large Datasets**

   When working with large datasets, use the `DataManager` for efficient chunking:

   ```python
   from patternforge.candlestick.visualization import DataManager
   data_manager = DataManager(df, chunk_size=1000)
   ```

4. **Visualization Performance**

   Enable caching for better performance when creating multiple visualizations:

   ```python
   visualizer.base_settings.cache = VisualizationCache(max_size=100)
   ```

## Contributing

To contribute to the candlestick visualization module:

1. Follow the existing code style and documentation patterns
2. Add proper docstrings to new functions and classes
3. Update this documentation with new features
4. Add tests for new functionality in the `tests/test_candlestick/test_visualization.py` file

## License

This module is part of the PatternForge package and is subject to the package's overall license.
