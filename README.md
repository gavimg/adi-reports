# Gadagi Reports - Analytics Micro-frontend

A dedicated micro-frontend for reporting, analytics, and data visualization within the Gadagi platform architecture.

## Overview

Gadagi Reports is a remote micro-frontend that provides comprehensive reporting capabilities, data visualization, and analytics features. It's designed to be loaded dynamically by the Gadagi Host application using Webpack Module Federation.

## Features

- 📊 **Data Visualization** - Charts, graphs, and dashboards
- 📈 **Analytics** - Business intelligence and insights
- 🗓️ **Date Range Filtering** - Flexible time period selection
- 📱 **Responsive Design** - Mobile-friendly interface
- 🎨 **Theme Integration** - Consistent with design system
- 📄 **Export Options** - PDF, Excel, CSV exports

## Architecture

```
┌─────────────────────────────────────┐
│            Gadagi Reports              │
│  ┌─────────────┬─────────────────┐  │
│  │ Dashboard   │ Chart Library   │  │
│  │ - KPIs      │ - Bar Charts    │  │
│  │ - Metrics   │ - Line Charts   │  │
│  │ - Trends    │ - Pie Charts    │  │
│  └─────────────┴─────────────────┘  │
│  ┌─────────────────────────────────┐  │
│  │      Report Builder            │  │
│  │  - Drag & Drop                 │  │
│  │  - Custom Filters              │  │
│  │  - Export Options              │  │
│  └─────────────────────────────────┘  │
└─────────────────────────────────────┘
```

## Quick Start

```bash
# Install dependencies
npm install

# Start development server
npm start

# Build for production
npm run build
```

## Development Server

- **Development URL**: http://localhost:3002
- **Remote Entry**: http://localhost:3002/remoteEntry.js

## Module Federation Configuration

### Remote Configuration

```javascript
// webpack.config.js
const ModuleFederationPlugin = require('@module-federation/enhanced');

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'adiReports',
      filename: 'remoteEntry.js',
      exposes: {
        './ReportsApp': './src/ReportsApp.tsx',
      },
      shared: {
        react: { singleton: true },
        'react-dom': { singleton: true },
        '@gadagi/types': { singleton: true },
        '@gadagi/design-system': { singleton: true },
      },
    }),
  ],
};
```

### Exposed Module

```typescript
// src/ReportsApp.tsx
import React from 'react';
import { Dashboard, ReportBuilder, ChartLibrary } from './components';

const ReportsApp: React.FC = () => {
  return (
    <div style={{ padding: '2rem', background: '#f0fdf4', minHeight: '100vh' }}>
      <h1>Reports Module</h1>
      <p>This is the adi-reports micro-frontend.</p>
      
      {/* Reporting Components */}
      <Dashboard />
      <ReportBuilder />
      <ChartLibrary />
    </div>
  );
};

export default ReportsApp;
```

## Components

### Dashboard

Main dashboard with KPIs and metrics.

```tsx
import { Dashboard } from './components/Dashboard';

<Dashboard
  metrics={dashboardMetrics}
  timeRange={selectedTimeRange}
  onTimeRangeChange={handleTimeRangeChange}
/>
```

**Props:**
- `metrics: DashboardMetrics[]` - Array of dashboard metrics
- `timeRange: TimeRange` - Selected time range
- `onTimeRangeChange: (range: TimeRange) => void` - Time range change handler
- `loading?: boolean` - Loading state

### ChartLibrary

Reusable chart components.

```tsx
import { BarChart, LineChart, PieChart } from './components/ChartLibrary';

<BarChart
  data={salesData}
  title="Sales by Month"
  xAxis="month"
  yAxis="sales"
/>

<LineChart
  data={revenueData}
  title="Revenue Trend"
  xAxis="date"
  yAxis="revenue"
/>

<PieChart
  data={categoryData}
  title="Category Distribution"
  valueField="value"
  labelField="category"
/>
```

### ReportBuilder

Interactive report creation tool.

```tsx
import { ReportBuilder } from './components/ReportBuilder';

<ReportBuilder
  onSave={handleSaveReport}
  onCancel={handleCancel}
  initialData={reportData}
/>
```

**Props:**
- `onSave: (report: ReportConfig) => void` - Save handler
- `onCancel: () => void` - Cancel handler
- `initialData?: ReportConfig` - Initial report data

## Data Management

### Report Service

```typescript
// src/services/reportService.ts
import { ApiResponse } from '@gadagi/types';

export interface ReportData {
  id: string;
  title: string;
  type: 'chart' | 'table' | 'dashboard';
  config: ReportConfig;
  createdAt: string;
  updatedAt: string;
}

export class ReportService {
  // Get all reports
  static async getReports(): Promise<ReportData[]> {
    const response = await fetch('/api/reports');
    return response.json();
  }

  // Get report by ID
  static async getReport(id: string): Promise<ReportData> {
    const response = await fetch(`/api/reports/${id}`);
    return response.json();
  }

  // Create report
  static async createReport(report: Partial<ReportData>): Promise<ReportData> {
    const response = await fetch('/api/reports', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(report),
    });
    return response.json();
  }

  // Get chart data
  static async getChartData(reportId: string, filters?: any): Promise<any> {
    const params = new URLSearchParams(filters);
    const response = await fetch(`/api/reports/${reportId}/data?${params}`);
    return response.json();
  }

  // Export report
  static async exportReport(reportId: string, format: 'pdf' | 'excel' | 'csv'): Promise<Blob> {
    const response = await fetch(`/api/reports/${reportId}/export?format=${format}`);
    return response.blob();
  }
}
```

### State Management

```tsx
// src/hooks/useReports.ts
import { useState, useEffect } from 'react';
import { ReportData } from '../services/reportService';

export const useReports = () => {
  const [reports, setReports] = useState<ReportData[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const fetchReports = async () => {
    setLoading(true);
    try {
      const data = await ReportService.getReports();
      setReports(data);
    } catch (err) {
      setError('Failed to fetch reports');
    } finally {
      setLoading(false);
    }
  };

  const createReport = async (reportData: Partial<ReportData>) => {
    try {
      const newReport = await ReportService.createReport(reportData);
      setReports(prev => [...prev, newReport]);
      return newReport;
    } catch (err) {
      setError('Failed to create report');
      throw err;
    }
  };

  useEffect(() => {
    fetchReports();
  }, []);

  return {
    reports,
    loading,
    error,
    createReport,
    refetch: fetchReports,
  };
};
```

## Features

### Time Range Selection

```tsx
// src/components/TimeRangeSelector.tsx
import { Button } from '@gadagi/design-system';

interface TimeRangeSelectorProps {
  value: TimeRange;
  onChange: (range: TimeRange) => void;
}

const TimeRangeSelector: React.FC<TimeRangeSelectorProps> = ({ value, onChange }) => {
  const ranges = [
    { label: 'Today', value: 'today' },
    { label: 'This Week', value: 'week' },
    { label: 'This Month', value: 'month' },
    { label: 'This Quarter', value: 'quarter' },
    { label: 'This Year', value: 'year' },
    { label: 'Custom', value: 'custom' },
  ];

  return (
    <div className="time-range-selector">
      {ranges.map(range => (
        <Button
          key={range.value}
          variant={value === range.value ? 'primary' : 'secondary'}
          onClick={() => onChange(range.value as TimeRange)}
        >
          {range.label}
        </Button>
      ))}
    </div>
  );
};
```

### Chart Types

```tsx
// src/components/charts/BarChart.tsx
import React from 'react';
import { Bar } from 'react-chartjs-2';

interface BarChartProps {
  data: any[];
  title: string;
  xAxis: string;
  yAxis: string;
  color?: string;
}

const BarChart: React.FC<BarChartProps> = ({ 
  data, 
  title, 
  xAxis, 
  yAxis, 
  color = '#4a3fb5' 
}) => {
  const chartData = {
    labels: data.map(item => item[xAxis]),
    datasets: [
      {
        label: title,
        data: data.map(item => item[yAxis]),
        backgroundColor: color,
        borderColor: color,
        borderWidth: 1,
      },
    ],
  };

  const options = {
    responsive: true,
    plugins: {
      legend: {
        position: 'top' as const,
      },
      title: {
        display: true,
        text: title,
      },
    },
  };

  return <Bar data={chartData} options={options} />;
};
```

### Export Functionality

```tsx
// src/components/ExportButton.tsx
import { Button } from '@gadagi/design-system';
import { ReportService } from '../services/reportService';

interface ExportButtonProps {
  reportId: string;
  format: 'pdf' | 'excel' | 'csv';
  filename?: string;
}

const ExportButton: React.FC<ExportButtonProps> = ({ 
  reportId, 
  format, 
  filename = `report-${reportId}` 
}) => {
  const [exporting, setExporting] = useState(false);

  const handleExport = async () => {
    setExporting(true);
    try {
      const blob = await ReportService.exportReport(reportId, format);
      const url = window.URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = `${filename}.${format}`;
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
      window.URL.revokeObjectURL(url);
    } catch (error) {
      console.error('Export failed:', error);
    } finally {
      setExporting(false);
    }
  };

  return (
    <Button
      onClick={handleExport}
      disabled={exporting}
      variant="secondary"
    >
      {exporting ? 'Exporting...' : `Export ${format.toUpperCase()}`}
    </Button>
  );
};
```

## Integration

### Host Integration

The ReportsApp component is designed to be loaded by the ADI Host:

```typescript
// In ADI Host
const ReportsApp = React.lazy(() => import('adiReports/ReportsApp'));

// Route configuration
<Route path="/reports" element={
  <Suspense fallback={<div>Loading Reports...</div>}>
    <ReportsApp />
  </Suspense>
} />
```

### Shared Dependencies

The micro-frontend shares dependencies with the host:

- React & React DOM
- @gadagi/types
- @gadagi/design-system
- Chart.js libraries

## Styling

### Theme Integration

```tsx
// Use design system tokens
import { colors, spacing } from '@gadagi/design-system';

const reportStyles = {
  container: {
    padding: spacing[4],
    backgroundColor: colors.neutral[50],
  },
  chart: {
    backgroundColor: colors.neutral[100],
    borderRadius: '8px',
    padding: spacing[3],
  },
};
```

### Custom Styles

```css
/* src/styles/Reports.css */
.reports-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 1rem;
}

.chart-card {
  background: white;
  border-radius: 8px;
  padding: 1.5rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.chart-card:hover {
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
}

.dashboard-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
  margin-bottom: 2rem;
}

.metric-card {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 1.5rem;
  border-radius: 8px;
  text-align: center;
}
```

## Testing

### Unit Tests

```bash
# Run tests
npm test

# Run with coverage
npm run test:coverage
```

### Component Tests

```typescript
// src/components/__tests__/Dashboard.test.tsx
import { render, screen } from '@testing-library/react';
import { Dashboard } from '../Dashboard';

describe('Dashboard', () => {
  const mockMetrics = [
    { id: '1', title: 'Total Users', value: 1000, change: 10 },
    { id: '2', title: 'Revenue', value: 50000, change: 15 },
  ];

  it('renders dashboard metrics', () => {
    render(<Dashboard metrics={mockMetrics} />);
    expect(screen.getByText('Total Users')).toBeInTheDocument();
    expect(screen.getByText('Revenue')).toBeInTheDocument();
  });

  it('displays metric values correctly', () => {
    render(<Dashboard metrics={mockMetrics} />);
    expect(screen.getByText('1000')).toBeInTheDocument();
    expect(screen.getByText('50000')).toBeInTheDocument();
  });
});
```

## Performance Optimization

### Chart Optimization

```tsx
// Memoize chart components
import React, { memo } from 'react';

const OptimizedBarChart = memo(({ data, ...props }) => {
  return <BarChart data={data} {...props} />;
});
```

### Data Caching

```tsx
// src/hooks/useReportData.ts
import { useQuery } from 'react-query';

export const useReportData = (reportId: string, filters?: any) => {
  return useQuery(
    ['reportData', reportId, filters],
    () => ReportService.getChartData(reportId, filters),
    {
      staleTime: 5 * 60 * 1000, // 5 minutes
      cacheTime: 10 * 60 * 1000, // 10 minutes
    }
  );
};
```

## Deployment

### Environment Configuration

```typescript
// config/environment.ts
export const config = {
  apiUrl: process.env.REACT_APP_API_URL || 'http://localhost:4000',
  chartApiUrl: process.env.REACT_APP_CHART_API_URL || 'http://localhost:4001',
  environment: process.env.NODE_ENV || 'development',
};
```

### Build Configuration

```bash
# Production build
npm run build

# Preview build
npm run preview
```

## Troubleshooting

### Common Issues

1. **Chart Rendering Issues**
   - Check Chart.js dependencies
   - Verify data format
   - Check canvas rendering context

2. **Export Failures**
   - Verify API endpoints
   - Check file format support
   - Ensure proper headers

3. **Performance Issues**
   - Implement data pagination
   - Use chart optimization
   - Add loading states

## Contributing

1. Follow component patterns
2. Add tests for new charts
3. Update documentation
4. Ensure accessibility compliance

## License

MIT © Gadagi Team
