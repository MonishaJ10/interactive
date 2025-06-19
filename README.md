<div *ngIf="chartOptions">
  <apx-chart
    [series]="chartOptions.series"
    [chart]="chartOptions.chart"
    [xaxis]="chartOptions.xaxis"
    [labels]="chartOptions.labels"
    [title]="chartOptions.title">
  </apx-chart>
</div>







import { Component, Input, OnChanges, SimpleChanges } from '@angular/core';
import { CommonModule } from '@angular/common';
import { NgApexchartsModule } from 'ng-apexcharts';
import {
  ApexChart,
  ApexNonAxisChartSeries,
  ApexTitleSubtitle,
  ApexAxisChartSeries,
  ApexXAxis
} from 'ng-apexcharts';

import { Dashboardd } from '../dashboard.model';
import { DashboardService } from '../dashboard.service';

@Component({
  selector: 'app-chart-viewer',
  standalone: true,
  imports: [CommonModule, NgApexchartsModule],
  templateUrl: './chart-viewer.component.html',
  styleUrls: ['./chart-viewer.component.css']
})
export class ChartViewerComponent implements OnChanges {
  @Input() dashboard: Dashboardd | null = null;

  chartOptions: {
    series: ApexAxisChartSeries | ApexNonAxisChartSeries;
    chart: ApexChart;
    xaxis?: ApexXAxis;
    title: ApexTitleSubtitle;
    labels?: string[];
  } = {
    series: [],
    chart: {
      type: 'bar',
      height: 350
    },
    title: {
      text: ''
    }
  };

  constructor(private dashboardService: DashboardService) {}

  ngOnChanges(changes: SimpleChanges): void {
    if (changes['dashboard'] && this.dashboard) {
      console.log('📊 Dashboard input changed:', this.dashboard);
      this.loadChartData();
    }
  }

  loadChartData(): void {
    if (!this.dashboard) return;

    const {
      model,
      groupBy,
      aggregation,
      aggregationField,
      chartType = 'bar',
      name
    } = this.dashboard;

    console.log('🟨 Fetching chart data for type:', chartType);
    console.log('🔹 Model:', model);
    console.log('🔹 GroupBy:', groupBy);
    console.log('🔹 Aggregation:', aggregation);
    console.log('🔹 AggregationField:', aggregationField);

    if (chartType === 'bar') {
      this.dashboardService.getBarChartData(model, groupBy, aggregation, aggregationField)
        .subscribe({
          next: (data: { LABEL: string; VALUE: number }[]) => {
            const chartData = data.map(d => ({ label: d.LABEL, value: d.VALUE }));
            console.log('✅ Bar chart data received:', chartData);

            this.chartOptions = {
              series: [{
                name: 'Value',
                data: chartData.map(d => d.value)
              }],
              chart: {
                type: 'bar',
                height: 350
              },
              xaxis: {
                categories: chartData.map(d => d.label)
              },
              title: {
                text: name || 'Bar Chart'
              }
            };
          },
          error: err => console.error('❌ Error fetching bar chart data:', err)
        });

    } else if (chartType === 'pie') {
      this.dashboardService.getPieChartData(model, groupBy, aggregation, aggregationField)
        .subscribe({
          next: (data: { LABEL: string; VALUE: number }[]) => {
            const chartData = data.map(d => ({ label: d.LABEL, value: d.VALUE }));
            console.log('✅ Pie chart data received:', chartData);

            this.chartOptions = {
              series: chartData.map(d => d.value),
              chart: {
                type: 'pie',
                height: 350
              },
              labels: chartData.map(d => d.label),
              title: {
                text: name || 'Pie Chart'
              }
            };
          },
          error: err => console.error('❌ Error fetching pie chart data:', err)
        });

    } else {
      console.warn('⚠️ Unknown chart type:', chartType);
    }
  }
}





















import { Component, Input, OnChanges, SimpleChanges } from '@angular/core';
import { CommonModule } from '@angular/common';
import { NgApexchartsModule } from 'ng-apexcharts';
import {
  ApexChart,
  ApexNonAxisChartSeries,
  ApexTitleSubtitle,
  ApexAxisChartSeries,
  ApexXAxis
} from 'ng-apexcharts';
import { Dashboardd } from '../dashboard.model';
import { DashboardService } from '../dashboard.service';

@Component({
  selector: 'app-chart-viewer',
  standalone: true,
  imports: [CommonModule, NgApexchartsModule],
  templateUrl: './chart-viewer.component.html',
  styleUrls: ['./chart-viewer.component.css']
})
export class ChartViewerComponent implements OnChanges {

  @Input() dashboard: Dashboardd | null = null;

  chartOptions: {
    series: ApexAxisChartSeries | ApexNonAxisChartSeries;
    chart: ApexChart;
    xaxis?: ApexXAxis;
    title: ApexTitleSubtitle;
    labels?: string[];
  } = {
    series: [],
    chart: {
      type: 'bar',
      height: 350
    },
    title: {
      text: ''
    },
    xaxis: {
      categories: []
    },
    labels: []
  };

  constructor(private dashboardService: DashboardService) {}

  ngOnChanges(changes: SimpleChanges): void {
    if (changes['dashboard'] && this.dashboard) {
      console.log('📥 Dashboard input changed:', this.dashboard);
      this.loadChartData();
    }
  }

  loadChartData(): void {
    if (!this.dashboard) return;

    const {
      model,
      groupBy,
      aggregation,
      aggregationField,
      name,
      chartType
    } = this.dashboard;

    const type = chartType?.toLowerCase() || 'bar';
    console.log('📊 Fetching chart data for type:', type);
    console.log('📌 Params:', { model, groupBy, aggregation, aggregationField });

    if (type === 'bar') {
      this.dashboardService.getBarChartData(model, groupBy, aggregation, aggregationField)
        .subscribe({
          next: (data: { LABEL: string; VALUE: number }[]) => {
            const chartData = data.map(item => ({
              label: item.LABEL,
              value: item.VALUE
            }));
            console.log('✅ Bar chart data received:', chartData);

            this.chartOptions = {
              series: [{
                name: 'Value',
                data: chartData.map(d => d.value)
              }],
              chart: {
                type: 'bar',
                height: 350
              },
              xaxis: {
                categories: chartData.map(d => d.label)
              },
              title: {
                text: name || 'Bar Chart'
              },
              labels: []
            };
          },
          error: (err) => console.error('❌ Error fetching bar chart data:', err)
        });

    } else if (type === 'pie') {
      this.dashboardService.getPieChartData(model, groupBy, aggregation, aggregationField)
        .subscribe({
          next: (data: { LABEL: string; VALUE: number }[]) => {
            const chartData = data.map(item => ({
              label: item.LABEL,
              value: item.VALUE
            }));
            console.log('✅ Pie chart data received:', chartData);

            this.chartOptions = {
              series: chartData.map(d => d.value),
              chart: {
                type: 'pie',
                height: 350
              },
              title: {
                text: name || 'Pie Chart'
              },
              labels: chartData.map(d => d.label)
            };
          },
          error: (err) => console.error('❌ Error fetching pie chart data:', err)
        });

    } else {
      console.warn('⚠️ Unknown chart type:', chartType);
    }
  }
}




<apx-chart
  [series]="chartOptions.series"
  [chart]="chartOptions.chart"
  [xaxis]="chartOptions.xaxis"
  [labels]="chartOptions.labels"
  [title]="chartOptions.title">
</apx-chart>





















import { Component, Input, OnChanges } from '@angular/core';
import { ApexChart, ApexNonAxisChartSeries, ApexAxisChartSeries, ApexXAxis, ApexTitleSubtitle } from 'ng-apexcharts';
import { Dashboardd } from '../dashboard.model';
import { DashboardService } from '../dashboard.service';

@Component({
  selector: 'app-chart-viewer',
  templateUrl: './chart-viewer.component.html',
  styleUrls: ['./chart-viewer.component.css']
})
export class ChartViewerComponent implements OnChanges {
  @Input() dashboard: Dashboardd | null = null;

  chartOptions: any = {};
  chartType: 'bar' | 'pie' = 'bar';

  constructor(private dashboardService: DashboardService) {}

  ngOnChanges(): void {
    if (!this.dashboard) return;

    const { chartType, model, groupBy, aggregation, aggregationField, name } = this.dashboard;

    console.log('📊 Dashboard input changed:', this.dashboard);
    console.log('🟡 Fetching chart data for type:', chartType);

    if (chartType === 'bar') {
      this.chartType = 'bar';
      this.dashboardService
        .getBarChartData(model, groupBy, aggregation, aggregationField)
        .subscribe({
          next: (data) => {
            console.log('✅ Bar chart data received:', data);

            const barData = data.map((item: any) => ({
              label: item.LABEL,
              value: item.VALUE
            }));

            this.chartOptions = {
              series: [{
                name: 'Value',
                data: barData.map(d => d.value)
              }],
              chart: {
                type: 'bar',
                height: 350
              },
              xaxis: {
                categories: barData.map(d => d.label)
              },
              title: {
                text: name || 'Bar Chart'
              }
            };
          },
          error: (err) => {
            console.error('❌ Error fetching bar chart data:', err);
          }
        });

    } else if (chartType === 'pie') {
      this.chartType = 'pie';
      this.dashboardService
        .getPieChartData(model, groupBy, aggregation, aggregationField)
        .subscribe({
          next: (data) => {
            console.log('✅ Pie chart data received:', data);

            const pieData = data.map((item: any) => ({
              label: item.LABEL,
              value: item.VALUE
            }));

            this.chartOptions = {
              series: pieData.map(d => d.value),
              chart: {
                type: 'pie',
                height: 350
              },
              labels: pieData.map(d => d.label),
              title: {
                text: name || 'Pie Chart'
              }
            };
          },
          error: (err) => {
            console.error('❌ Error fetching pie chart data:', err);
          }
        });
    } else {
      console.warn('⚠️ Unknown chart type:', chartType);
    }
  }
}


<div *ngIf="dashboard">
  <h3 style="text-align: center;">
    {{ dashboard.name || 'Chart Viewer' }}
  </h3>

  <div *ngIf="chartType === 'bar'">
    <apx-chart
      [series]="chartOptions.series"
      [chart]="chartOptions.chart"
      [xaxis]="chartOptions.xaxis"
      [title]="chartOptions.title">
    </apx-chart>
  </div>

  <div *ngIf="chartType === 'pie'">
    <apx-chart
      [series]="chartOptions.series"
      [chart]="chartOptions.chart"
      [labels]="chartOptions.labels"
      [title]="chartOptions.title">
    </apx-chart>
  </div>
</div>

<div *ngIf="!dashboard">
  <p style="text-align: center; color: red;">⚠️ No dashboard selected to render chart.</p>
</div>