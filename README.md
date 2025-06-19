import { Component, Input, OnChanges, SimpleChanges } from '@angular/core';
import {
  ApexChart,
  ApexAxisChartSeries,
  ApexNonAxisChartSeries,
  ApexXAxis,
  ApexTitleSubtitle
} from 'ng-apexcharts';
import { Dashboardd } from '../dashboard.model';
import { DashboardService } from '../dashboard.service';
import { CommonModule } from '@angular/common';
import { NgApexchartsModule } from 'ng-apexcharts';

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
    title?: ApexTitleSubtitle;
    labels?: string[];
  } = {
    series: [],
    chart: { type: 'bar', height: 350 },
    xaxis: { categories: [] },
    title: { text: '' },
    labels: []
  };

  constructor(private dashboardService: DashboardService) {}

  ngOnChanges(changes: SimpleChanges): void {
    if (changes['dashboard'] && this.dashboard) {
      this.loadChartData();
    }
  }

  loadChartData(): void {
    if (!this.dashboard) return;

    const { model, groupBy, aggregation, aggregationField, chartType, name } = this.dashboard;

    if (chartType === 'BarChart') {
      this.dashboardService.getBarChartData(model, groupBy, aggregation, aggregationField).subscribe({
        next: (data) => {
          this.chartOptions = {
            series: [{ name: 'Value', data: data.map((d: any) => d.value) }],
            chart: { type: 'bar', height: 350 },
            xaxis: { categories: data.map((d: any) => d.label) },
            title: { text: name ?? 'Bar Chart' }
          };
        },
        error: (err) => console.error('Error fetching bar chart data:', err)
      });
    } else if (chartType === 'PieChart') {
      this.dashboardService.getPieChartData(model, groupBy, aggregation, aggregationField).subscribe({
        next: (data) => {
          this.chartOptions = {
            series: data.map((d: any) => d.value),
            chart: { type: 'pie', height: 350 },
            labels: data.map((d: any) => d.label),
            xaxis: { categories: [] },
            title: { text: name ?? 'Pie Chart' }
          };
        },
        error: (err) => console.error('Error fetching pie chart data:', err)
      });
    } else {
      console.warn('Unknown chart type:', chartType);
    }
  }
}


<div *ngIf="chartOptions.chart">
  <apx-chart
    [series]="chartOptions.series"
    [chart]="chartOptions.chart"
    *ngIf="chartOptions.chart.type === 'bar'; else pieChart"
    [xaxis]="chartOptions.xaxis || {}"
    [title]="chartOptions.title || { text: '' }">
  </apx-chart>

  <ng-template #pieChart>
    <apx-chart
      [series]="chartOptions.series"
      [chart]="chartOptions.chart"
      [labels]="chartOptions.labels || []"
      [title]="chartOptions.title || { text: '' }">
    </apx-chart>
  </ng-template>
</div>






















import { Component, Input, OnChanges, SimpleChanges } from '@angular/core';
import { CommonModule } from '@angular/common';
import {
  ApexChart,
  ApexAxisChartSeries,
  ApexNonAxisChartSeries,
  ApexXAxis,
  ApexTitleSubtitle
} from 'ng-apexcharts';
import { NgApexchartsModule } from 'ng-apexcharts';
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
    labels: string[];
  } = {
    series: [],
    chart: {
      type: 'bar',
      height: 350
    },
    xaxis: {
      categories: []
    },
    title: {
      text: ''
    },
    labels: []
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
      chartType,
      name
    } = this.dashboard;

    console.log('🔍 Fetching chart data for type:', chartType);

    if (chartType === 'BarChart') {
      this.dashboardService
        .getBarChartData(model, groupBy, aggregation, aggregationField)
        .subscribe({
          next: (data) => {
            console.log('📦 Bar chart data received:', data);
            this.chartOptions = {
              series: [{ name: 'Value', data: data.map((d: any) => d.value) }],
              chart: { type: 'bar', height: 350 },
              xaxis: { categories: data.map((d: any) => d.label) },
              title: { text: name || 'Bar Chart' },
              labels: []
            };
          },
          error: (err) => console.error('❌ Error fetching bar chart data:', err)
        });
    } else if (chartType === 'PieChart') {
      this.dashboardService
        .getPieChartData(model, groupBy, aggregation, aggregationField)
        .subscribe({
          next: (data) => {
            console.log('✅ Pie chart data received:', data);
            this.chartOptions = {
              series: data.map((d: any) => d.value),
              chart: { type: 'pie', height: 350 },
              labels: data.map((d: any) => d.label),
              title: { text: name || 'Pie Chart' },
              xaxis: { categories: [] } // just to satisfy type safety
            };
          },
          error: (err) => console.error('❌ Error fetching pie chart data:', err)
        });
    } else {
      console.warn('⚠️ Unknown chart type:', chartType);
    }
  }
}







import { Component, Input, OnChanges, SimpleChanges } from '@angular/core';
import { CommonModule } from '@angular/common';
import {
  ApexChart,
  ApexAxisChartSeries,
  ApexNonAxisChartSeries,
  ApexXAxis,
  ApexTitleSubtitle
} from 'ng-apexcharts';
import { NgApexchartsModule } from 'ng-apexcharts';
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
    title?: ApexTitleSubtitle;
    labels?: string[];
  } = {
    series: [],
    chart: {
      type: 'bar',
      height: 350
    },
    xaxis: {
      categories: []
    },
    title: {
      text: ''
    },
    labels: []
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
      chartType,
      name
    } = this.dashboard;

    console.log('🔍 Fetching chart data for type:', chartType);

    if (chartType === 'BarChart') {
      this.dashboardService
        .getBarChartData(model, groupBy, aggregation, aggregationField)
        .subscribe({
          next: (data) => {
            console.log('📦 Bar chart data received:', data);
            this.chartOptions = {
              series: [{ name: 'Value', data: data.map((d: any) => d.value) }],
              chart: { type: 'bar', height: 350 },
              xaxis: { categories: data.map((d: any) => d.label) },
              title: { text: name ?? 'Bar Chart' },
              labels: []
            };
          },
          error: (err) => console.error('❌ Error fetching bar chart data:', err)
        });
    } else if (chartType === 'PieChart') {
      this.dashboardService
        .getPieChartData(model, groupBy, aggregation, aggregationField)
        .subscribe({
          next: (data) => {
            console.log('✅ Pie chart data received:', data);
            this.chartOptions = {
              series: data.map((d: any) => d.value),
              chart: { type: 'pie', height: 350 },
              labels: data.map((d: any) => d.label),
              title: { text: name ?? 'Pie Chart' }
            };
          },
          error: (err) => console.error('❌ Error fetching pie chart data:', err)
        });
    } else {
      console.warn('⚠️ Unknown chart type:', chartType);
    }
  }
}

<div *ngIf="!dashboard">
  <p style="color: red">⚠️ No dashboard input received</p>
</div>

<div *ngIf="dashboard">
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
  ApexAxisChartSeries,
  ApexNonAxisChartSeries,
  ApexXAxis,
  ApexTitleSubtitle
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
    title?: ApexTitleSubtitle;
    labels?: string[];
  } = {
    series: [],
    chart: { type: 'bar', height: 350 },
    xaxis: { categories: [] },
    title: { text: '' },
    labels: []
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

    const { model, groupBy, aggregation, aggregationField, chartType, name } = this.dashboard;

    console.log('🔍 Fetching chart data for type:', chartType);

    if (chartType === 'BarChart') {
      this.dashboardService.getBarChartData(model, groupBy, aggregation, aggregationField).subscribe({
        next: (data) => {
          console.log('📦 Bar chart data received:', data);
          this.chartOptions = {
            series: [{ name: 'Value', data: data.map(d => d.value) }],
            chart: { type: 'bar', height: 350 },
            xaxis: { categories: data.map(d => d.label) },
            title: { text: name || 'Bar Chart' },
          };
        },
        error: (err) => console.error('❌ Error fetching bar chart data:', err)
      });
    } else if (chartType === 'PieChart') {
      this.dashboardService.getPieChartData(model, groupBy, aggregation, aggregationField).subscribe({
        next: (data) => {
          console.log('✅ Pie chart data received:', data);
          this.chartOptions = {
            series: data.map(d => d.value),
            chart: { type: 'pie', height: 350 },
            labels: data.map(d => d.label),
            title: { text: name || 'Pie Chart' },
          };
        },
        error: (err) => console.error('❌ Error fetching pie chart data:', err)
      });
    } else {
      console.warn('⚠️ Unknown chart type:', chartType);
    }
  }
}

<div *ngIf="!dashboard">
  <p style="color: red">⚠️ No dashboard input received</p>
</div>

<div *ngIf="dashboard">
  <apx-chart
    [series]="chartOptions.series"
    [chart]="chartOptions.chart"
    [xaxis]="chartOptions.xaxis"
    [labels]="chartOptions.labels"
    [title]="chartOptions.title">
  </apx-chart>
</div>























package com.example.recon_connect.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Map;

@Service
public class ChartDataService {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    public List<Map<String, Object>> getChartData(String model, String groupBy, String aggregation, String aggregationField) {
        // Validate aggregation type
        String aggregationFunction = switch (aggregation.toLowerCase()) {
            case "count" -> "COUNT(*)";
            case "sum" -> "SUM(" + aggregationField + ")";
            case "avg" -> "AVG(" + aggregationField + ")";
            default -> throw new IllegalArgumentException("Invalid aggregation: " + aggregation);
        };

        // Construct dynamic SQL query
        String query = String.format(
            "SELECT %s AS label, %s AS value FROM %s WHERE %s IS NOT NULL GROUP BY %s",
            groupBy, aggregationFunction, model, groupBy, groupBy
        );

        // Log for debug
        System.out.println("Executing query: " + query);

        return jdbcTemplate.queryForList(query);
    }
}

package com.example.recon_connect.controller;

import com.example.recon_connect.service.ChartDataService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;
import java.util.Map;

@CrossOrigin(origins = "http://localhost:4200")
@RestController
@RequestMapping("/api")
public class ChartDataController {

    @Autowired
    private ChartDataService chartDataService;

    @GetMapping("/bar-chart-data")
    public ResponseEntity<List<Map<String, Object>>> getBarChartData(
            @RequestParam String model,
            @RequestParam String groupBy,
            @RequestParam String aggregation,
            @RequestParam String aggregationField
    ) {
        List<Map<String, Object>> result = chartDataService.getChartData(model, groupBy, aggregation, aggregationField);
        return ResponseEntity.ok(result);
    }

    @GetMapping("/pie-chart-data")
    public ResponseEntity<List<Map<String, Object>>> getPieChartData(
            @RequestParam String model,
            @RequestParam String groupBy,
            @RequestParam String aggregation,
            @RequestParam String aggregationField
    ) {
        List<Map<String, Object>> result = chartDataService.getChartData(model, groupBy, aggregation, aggregationField);
        return ResponseEntity.ok(result);
    }
}