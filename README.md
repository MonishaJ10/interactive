<div *ngIf="isBarChart || isPieChart">
  <apx-chart
    *ngIf="chartOptions.series"
    [series]="chartOptions.series"
    [chart]="chartOptions.chart"
    [xaxis]="chartOptions.xaxis"
    [labels]="chartOptions.labels"
    [dataLabels]="chartOptions.dataLabels"
    [plotOptions]="chartOptions.plotOptions"
    [responsive]="chartOptions.responsive"
    [title]="chartOptions.title"
  ></apx-chart>
</div>











import { Component, Input, OnChanges, SimpleChanges } from '@angular/core';
import { ApexChart, ApexNonAxisChartSeries, ApexResponsive, ApexTitleSubtitle, ApexAxisChartSeries, ApexXAxis, ApexDataLabels, ApexPlotOptions } from 'ng-apexcharts';
import { Dashboardd } from '../dashboard.model';
import { DashboardService } from '../dashboard.service';

export type ChartOptions = {
  series: ApexAxisChartSeries | ApexNonAxisChartSeries;
  chart: ApexChart;
  xaxis?: ApexXAxis;
  dataLabels?: ApexDataLabels;
  plotOptions?: ApexPlotOptions;
  labels?: string[];
  title?: ApexTitleSubtitle;
  responsive?: ApexResponsive[];
};

@Component({
  selector: 'app-chart-viewer',
  templateUrl: './chart-viewer.component.html',
  styleUrls: ['./chart-viewer.component.css']
})
export class ChartViewerComponent implements OnChanges {
  @Input() dashboard?: Dashboardd;
  chartOptions: Partial<ChartOptions> = {};
  isBarChart = false;
  isPieChart = false;

  constructor(private dashboardService: DashboardService) {}

  ngOnChanges(changes: SimpleChanges): void {
    if (this.dashboard) {
      const chartType = this.dashboard.chartType;

      // Normalize chartType for consistency
      const normalizedChartType = chartType === 'BarChart' ? 'bar' :
                                   chartType === 'PieChart' ? 'pie' : '';

      if (normalizedChartType) {
        this.loadChartData(normalizedChartType);
      } else {
        console.warn('Unknown chart type:', chartType);
      }
    }
  }

  loadChartData(type: string): void {
    this.isBarChart = type === 'bar';
    this.isPieChart = type === 'pie';

    if (type === 'bar') {
      this.dashboardService.getBarChartData(this.dashboard!).subscribe(data => {
        this.renderBarChart(data);
      });
    } else if (type === 'pie') {
      this.dashboardService.getPieChartData(this.dashboard!).subscribe(data => {
        this.renderPieChart(data);
      });
    }
  }

  renderBarChart(data: any[]): void {
    const categories = data.map(item => item.label);
    const seriesData = data.map(item => item.value);

    this.chartOptions = {
      series: [
        {
          name: this.dashboard?.title || 'Data',
          data: seriesData
        }
      ],
      chart: {
        type: 'bar',
        height: 350
      },
      xaxis: {
        categories
      },
      dataLabels: {
        enabled: true
      },
      title: {
        text: this.dashboard?.title || 'Bar Chart'
      }
    };
  }

  renderPieChart(data: any[]): void {
    const labels = data.map(item => item.label);
    const series = data.map(item => item.value);

    this.chartOptions = {
      series,
      chart: {
        type: 'pie',
        height: 350
      },
      labels,
      responsive: [
        {
          breakpoint: 480,
          options: {
            chart: {
              width: 320
            },
            legend: {
              position: 'bottom'
            }
          }
        }
      ],
      title: {
        text: this.dashboard?.title || 'Pie Chart'
      }
    };
  }
}











submitDashboard(): void {
  const uid = sessionStorage.getItem('uid') || 'unknown';

  // 🔁 Map lowercase chart type to API-compatible value
  const chartTypeForBackend = this.selectedChart === 'bar' ? 'BarChart' :
                              this.selectedChart === 'pie' ? 'PieChart' : 'BarChart';

  const dashboardPayload: Dashboardd = {
    name: this.formData.name,
    description: this.formData.description,
    isPublic: this.formData.isPublic,
    chartType: chartTypeForBackend, // ✅ mapped value
    model: this.model,
    groupBy: this.groupBy,
    aggregation: this.aggregation,
    aggregationField: this.aggregationField,
    createdBy: this.editData?.createdBy ?? uid,
    createdDate: this.editData?.createdDate ?? new Date().toISOString(),
    modifiedBy: this.isEditMode ? uid : undefined,
    modifiedDate: new Date().toISOString(),
    type: 'blank',
  };

  if (this.isEditMode && this.editData?.id) {
    this.dashboardService.updateDashboard(this.editData.id, dashboardPayload).subscribe({
      next: () => {
        console.log('Dashboard updated');
        this.dashboardUpdated.emit();
        this.dashboardClose.emit();
      },
      error: (err) => console.error('Update failed', err),
    });
  } else {
    this.dashboardService.addDashboard(dashboardPayload).subscribe({
      next: () => {
        console.log('Dashboard created');
        this.dashboardCreated.emit();
        this.dashboardClose.emit();
      },
      error: (err) => console.error('Create failed', err),
    });
  }
}



if (this.editData) {
  this.isEditMode = true;

  this.formData = {
    name: this.editData.name ?? '',
    description: this.editData.description ?? '',
    isPublic: this.editData.isPublic ?? false,
  };

  // 🔁 Convert chart type from backend format to UI format
  this.selectedChart = this.editData.chartType === 'BarChart' ? 'bar' :
                       this.editData.chartType === 'PieChart' ? 'pie' : 'bar';

  this.model = this.editData.model ?? '';
  this.groupBy = this.editData.groupBy ?? '';
  this.aggregation = this.editData.aggregation ?? '';
  this.aggregationField = this.editData.aggregationField ?? '';
}


















public Map<String, Object> getChartData(String model, String groupBy, String aggregation, String aggregationField) {
    String query = String.format(
        "SELECT %s AS label, %s(%s) AS value FROM %s GROUP BY %s",
        groupBy, aggregation, aggregationField, model, groupBy
    );

    List<Map<String, Object>> rows = jdbcTemplate.queryForList(query);

    List<String> categories = new ArrayList<>();
    List<Number> values = new ArrayList<>();

    for (Map<String, Object> row : rows) {
        categories.add(String.valueOf(row.get("label")));
        values.add((Number) row.get("value"));
    }

    Map<String, Object> result = new HashMap<>();
    result.put("categories", categories);
    result.put("values", values);
    return result;
}


@GetMapping("/bar-chart-data")
public ResponseEntity<Map<String, Object>> getBarChartData(
    @RequestParam String model,
    @RequestParam String groupBy,
    @RequestParam String aggregation,
    @RequestParam String aggregationField
) {
    Map<String, Object> result = chartDataService.getChartData(model, groupBy, aggregation, aggregationField);
    return ResponseEntity.ok(result);
}

@GetMapping("/pie-chart-data")
public ResponseEntity<Map<String, Object>> getPieChartData(
    @RequestParam String model,
    @RequestParam String groupBy,
    @RequestParam String aggregation,
    @RequestParam String aggregationField
) {
    Map<String, Object> result = chartDataService.getChartData(model, groupBy, aggregation, aggregationField);
    return ResponseEntity.ok(result);
}
















import { Component, Input, OnChanges, SimpleChanges } from '@angular/core';
import { DashboardService } from '../dashboard.service';
import { Dashboardd } from '../dashboard.model';
import {
  ApexChart,
  ApexAxisChartSeries,
  ApexNonAxisChartSeries,
  ApexXAxis,
  ApexTitleSubtitle
} from 'ng-apexcharts';
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
    xaxis: ApexXAxis;
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
              series: [{ name: 'Value', data: data.values }],
              chart: { type: 'bar', height: 350 },
              xaxis: { categories: data.categories },
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
              series: data.values,
              chart: { type: 'pie', height: 350 },
              xaxis: { categories: [] }, // Dummy to prevent undefined
              title: { text: name ?? 'Pie Chart' },
              labels: data.categories
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
import { NgApexchartsModule, ApexAxisChartSeries, ApexChart, ApexXAxis, ApexTitleSubtitle, ApexNonAxisChartSeries } from 'ng-apexcharts';
import { DashboardService } from '../dashboard.service';
import { Dashboardd } from '../dashboard.model';

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
    series?: ApexAxisChartSeries | ApexNonAxisChartSeries;
    chart?: ApexChart;
    xaxis?: ApexXAxis;
    title?: ApexTitleSubtitle;
    labels?: string[];
  } = {};

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
          console.log('✅ Bar chart data received:', data);
          this.chartOptions = {
            series: [{ name: 'Value', data: data.values }],
            chart: { type: 'bar', height: 350 },
            xaxis: { categories: data.categories },
            title: { text: name || 'Bar Chart' }
          };
        },
        error: (err) => console.error('❌ Error fetching bar chart data:', err)
      });
    } else if (chartType === 'PieChart') {
      this.dashboardService.getPieChartData(model, groupBy, aggregation, aggregationField).subscribe({
        next: (data) => {
          console.log('✅ Pie chart data received:', data);
          this.chartOptions = {
            series: data.values,
            chart: { type: 'pie', height: 350 },
            labels: data.categories,
            title: { text: name || 'Pie Chart' }
          };
        },
        error: (err) => console.error('❌ Error fetching pie chart data:', err)
      });
    } else {
      console.warn('⚠️ Unknown chart type:', chartType);
    }
  }
}
















<div *ngIf="selectedDashboardForView" class="popup-backdrop">
  <div class="popup-chart">
    <button class="close-btn" (click)="selectedDashboardForView = null">
      <mat-icon>close</mat-icon>
    </button>

    <!-- Debug block -->
    <div style="color: blue; font-size: 16px; margin-bottom: 1rem;">
      🧪 Dashboard: {{ selectedDashboardForView?.name }}
    </div>

    <!-- Chart viewer -->
    <app-chart-viewer [dashboard]="selectedDashboardForView"></app-chart-viewer>
  </div>
</div>



ngOnChanges(changes: SimpleChanges): void {
  console.log('📥 ngOnChanges:', changes);

  if (changes['dashboard']) {
    console.log('📊 New dashboard input:', this.dashboard);
    this.loadChartData();
  }
}























<ng-container *ngIf="dashboard; else noDashboard">
  <apx-chart
    *ngIf="chartOptions.series && chartOptions.chart"
    [series]="chartOptions.series"
    [chart]="chartOptions.chart"
    [xaxis]="chartOptions.xaxis"
    [labels]="chartOptions.labels"
    [title]="chartOptions.title">
  </apx-chart>
</ng-container>

<ng-template #noDashboard>
  <p style="color: red">⚠️ No dashboard input received</p>
</ng-template>







import { Component, Input, OnChanges, SimpleChanges } from '@angular/core';
import { DashboardService } from '../dashboard.service';
import { Dashboardd } from '../dashboard.model';
import {
  ApexAxisChartSeries,
  ApexChart,
  ApexXAxis,
  ApexTitleSubtitle,
  ApexNonAxisChartSeries
} from 'ng-apexcharts';

@Component({
  selector: 'app-chart-viewer',
  templateUrl: './chart-viewer.component.html',
  styleUrls: ['./chart-viewer.component.css']
})
export class ChartViewerComponent implements OnChanges {
  @Input() dashboard: Dashboardd | null = null;

  chartOptions: {
    series?: ApexAxisChartSeries | ApexNonAxisChartSeries;
    chart?: ApexChart;
    xaxis?: ApexXAxis;
    title?: ApexTitleSubtitle;
    labels?: string[];
  } = {};

  constructor(private dashboardService: DashboardService) {}

  ngOnChanges(changes: SimpleChanges): void {
    if (changes['dashboard'] && this.dashboard) {
      console.log('📊 Dashboard input changed:', this.dashboard);
      this.loadChartData();
    }
  }

  loadChartData(): void {
    if (!this.dashboard) return;

    const { model, groupBy, aggregation, aggregationField, chartType } = this.dashboard;
    console.log('📥 Fetching chart data for type:', chartType);

    if (chartType === 'BarChart') {
      this.dashboardService.getBarChartData(model, groupBy, aggregation, aggregationField).subscribe({
        next: (data) => {
          console.log('✅ Bar chart data received:', data);
          this.chartOptions = {
            series: [{ name: 'Value', data: data.values }],
            chart: { type: 'bar', height: 350 },
            xaxis: { categories: data.categories },
            title: { text: String(this.dashboard?.name || 'Bar Chart') }
          };
        },
        error: (err) => console.error('❌ Error fetching bar chart data:', err)
      });

    } else if (chartType === 'PieChart') {
      this.dashboardService.getPieChartData(model, groupBy, aggregation, aggregationField).subscribe({
        next: (data) => {
          console.log('✅ Pie chart data received:', data);
          this.chartOptions = {
            series: data.values,
            chart: { type: 'pie', height: 350 },
            labels: data.categories,
            title: { text: String(this.dashboard?.name || 'Pie Chart') }
          };
        },
        error: (err) => console.error('❌ Error fetching pie chart data:', err)
      });
    } else {
      console.warn('⚠️ Unknown chart type:', chartType);
    }
  }
}


{
  headerName: 'Name',
  field: 'name',
  cellRenderer: (params: any) => {
    return `<span class="dashboard-link">${params.value}</span>`;
  },
  onCellClicked: (params: any) => {
    this.onDashboardNameClick(params.data);
  }
}

















<div class="manage-dashboard-container" style="width: 90vw; overflow-x: auto;">
  <!-- Header -->
  <div class="header">
    <h1 class="title">Manage Dashboard</h1>
    <button class="close-btn" (click)="onClose()">
      <mat-icon>close</mat-icon>
    </button>
  </div>

  <!-- New Dashboard Buttons -->
  <div class="new-dashboard-section">
    <div class="dashboard-types">
      <div class="dashboard-card" (click)="toggleBlankDashboard()">
        <mat-icon class="dashboard-icon">speed</mat-icon>
        <div class="dashboard-name">Blank Dashboard</div>
      </div>
      <div class="dashboard-card" (click)="toggleInteractiveDashboard()">
        <mat-icon class="dashboard-icon">analytics</mat-icon>
        <div class="dashboard-name">Interactive Dashboard</div>
      </div>
    </div>
  </div>

  <!-- All Dashboards Table -->
  <div class="all-dashboards-section" *ngIf="!showBlankDashboard && !showInteractiveDashboard">
    <h2 class="section-title">All Dashboards</h2>
    <div class="table-container">
      <ag-grid-angular
        class="ag-theme-balham"
        style="width: 100%; height: 500px;"
        [gridOptions]="gridOptions"
        [rowData]="tableData"
        (gridReady)="onGridReady($event)">
      </ag-grid-angular>
    </div>
  </div>

  <!-- Chart Popup -->
  <div *ngIf="selectedDashboardForView" class="popup-backdrop">
    <div class="popup-chart">
      <button class="close-btn" (click)="selectedDashboardForView = null">
        <mat-icon>close</mat-icon>
      </button>
      <app-chart-viewer [dashboard]="selectedDashboardForView"></app-chart-viewer>
    </div>
  </div>

  <!-- Blank Dashboard Modal -->
  <app-blank-dashboard
    *ngIf="showBlankDashboard"
    [editData]="selectedBlankDashboard"
    (dashboardClose)="onClose()"
    (dashboardCreated)="onDashboardCreated()"
    (dashboardUpdated)="onDashboardUpdated()">
  </app-blank-dashboard>

  <!-- Interactive Dashboard Modal -->
  <app-interactive-dashboard
    *ngIf="showInteractiveDashboard"
    [editData]="selectedInteractiveDashboard"
    (dashboardClose)="onClose()"
    (dashboardCreated)="onDashboardCreated()"
    (dashboardUpdated)="onDashboardUpdated()">
  </app-interactive-dashboard>
</div>











// manage-dashboard.component.ts

import {
  Component, OnInit, Output, EventEmitter, ChangeDetectorRef, Renderer2, ElementRef, CUSTOM_ELEMENTS_SCHEMA
} from '@angular/core';
import { CommonModule } from '@angular/common';
import { MatIconModule } from '@angular/material/icon';
import { MatButtonModule } from '@angular/material/button';
import { MatSnackBar, MatSnackBarConfig } from '@angular/material/snack-bar';
import { AgGridModule } from 'ag-grid-angular';
import { ColDef } from 'ag-grid-community';
import { NgApexchartsModule } from 'ng-apexcharts';

import { Dashboardd } from './dashboard.model';
import { DashboardService } from './dashboard.service';
import { BlankDashboardComponent } from './blank-dashboard/blank-dashboard.component';
import { InteractiveDashboardComponent } from './interactive-dashboard/interactive-dashboard.component';

@Component({
  selector: 'app-manage-dashboard',
  standalone: true,
  imports: [
    CommonModule,
    MatIconModule,
    MatButtonModule,
    AgGridModule,
    NgApexchartsModule,
    BlankDashboardComponent,
    InteractiveDashboardComponent
  ],
  templateUrl: './manage-dashboard.component.html',
  styleUrls: ['./manage-dashboard.component.css'],
  schemas: [CUSTOM_ELEMENTS_SCHEMA],
})
export class ManageDashboardComponent implements OnInit {
  tableData: Dashboardd[] = [];
  showBlankDashboard = false;
  showInteractiveDashboard = false;

  selectedBlankDashboard?: Dashboardd;
  selectedInteractiveDashboard?: Dashboardd;

  @Output() closeDashboard = new EventEmitter<void>();
  @Output() selectDashboard = new EventEmitter<string>();

  selectedDashboardForView: Dashboardd | null = null;
  chartOptions: any;

  columnDefs: ColDef[] = [
    {
      headerName: 'Name',
      field: 'name',
      cellRenderer: this.nameCellRenderer.bind(this),
    },
    { field: 'description', headerName: 'Description' },
    { field: 'createdBy', headerName: 'Created By' },
    { field: 'createdDate', headerName: 'Created Date', valueFormatter: this.dateFormatter },
    { field: 'modifiedBy', headerName: 'Modified By' },
    { field: 'modifiedDate', headerName: 'Modified Date', valueFormatter: this.dateFormatter },
    { field: 'isPublic', headerName: 'Public', valueFormatter: this.booleanFormatter },
    {
      headerName: 'Actions',
      cellRenderer: this.actionCellRenderer.bind(this),
    },
  ];

  defaultColDef: ColDef = {
    resizable: true,
    sortable: true,
    filter: true,
    flex: 1,
  };

  constructor(
    private dashboardService: DashboardService,
    private renderer: Renderer2,
    private elementRef: ElementRef,
    private changeDetectorRef: ChangeDetectorRef,
    private snackBar: MatSnackBar
  ) {}

  ngOnInit(): void {
    this.loadDashboards();
  }

  loadDashboards(): void {
    this.dashboardService.getDashboards().subscribe({
      next: (data: Dashboardd[]) => {
        this.tableData = data;
        this.changeDetectorRef.detectChanges();
      },
      error: (error) => {
        console.error('Error loading dashboards:', error);
      },
    });
  }

  onEditClicked(dashboard: Dashboardd): void {
    const isInteractive = !!(dashboard.model || dashboard.groupBy || dashboard.aggregation || dashboard.aggregationField);
    if (isInteractive) {
      this.selectedInteractiveDashboard = dashboard;
      this.showInteractiveDashboard = true;
      this.showBlankDashboard = false;
    } else {
      this.selectedBlankDashboard = dashboard;
      this.showBlankDashboard = true;
      this.showInteractiveDashboard = false;
    }
    this.changeDetectorRef.detectChanges();
  }

  onDeleteClicked(dashboard: Dashboardd): void {
    this.dashboardService.deleteDashboard(dashboard.id).subscribe({
      next: () => {
        this.showSnackBar('Dashboard deleted successfully');
        this.loadDashboards();
      },
      error: (error) => {
        console.error('Error deleting dashboard:', error);
      }
    });
  }

  onDashboardCreated(): void {
    this.showSnackBar('Dashboard created successfully');
    this.loadDashboards();
    this.onBackToManage();
  }

  onDashboardUpdated(): void {
    this.showSnackBar('Dashboard updated successfully');
    this.loadDashboards();
    this.onBackToManage();
  }

  onBackToManage(): void {
    this.showBlankDashboard = false;
    this.showInteractiveDashboard = false;
    this.selectedBlankDashboard = undefined;
    this.selectedInteractiveDashboard = undefined;
    this.changeDetectorRef.detectChanges();
  }

  onDashboardNameClicked(dashboard: Dashboardd): void {
    this.selectedDashboardForView = dashboard;
  }

  // Chart popup logic
  loadChartPopup(dashboard: Dashboardd): void {
    this.selectedDashboardForView = dashboard;

    const { model, groupBy, aggregation, aggregationField, chartType } = dashboard;

    if (chartType === 'BarChart') {
      this.dashboardService.getBarChartData(model, groupBy, aggregation, aggregationField).subscribe(data => {
        this.chartOptions = {
          series: [{ name: 'Value', data: data.map((d: any) => d.value) }],
          chart: { type: 'bar', height: 350 },
          xaxis: { categories: data.map((d: any) => d.category) },
          title: { text: String(dashboard.name || 'Bar Chart'), align: 'center' }
        };
        this.changeDetectorRef.detectChanges();
      });
    } else if (chartType === 'PieChart') {
      this.dashboardService.getPieChartData(model, groupBy, aggregation, aggregationField).subscribe(data => {
        this.chartOptions = {
          series: data.map((d: any) => d.value),
          chart: { type: 'pie', height: 350 },
          labels: data.map((d: any) => d.category),
          title: { text: String(dashboard.name || 'Pie Chart'), align: 'center' }
        };
        this.changeDetectorRef.detectChanges();
      });
    }
  }

  nameCellRenderer(params: any): HTMLElement {
    const div = this.renderer.createElement('div');
    const link = this.renderer.createElement('a');
    this.renderer.setStyle(link, 'cursor', 'pointer');
    this.renderer.appendChild(link, this.renderer.createText(params.value));
    this.renderer.listen(link, 'click', () => this.loadChartPopup(params.data));
    this.renderer.appendChild(div, link);
    return div;
  }

  actionCellRenderer(params: any): HTMLElement {
    const div = this.renderer.createElement('div');
    const editBtn = this.renderer.createElement('button');
    const delBtn = this.renderer.createElement('button');

    const editIcon = this.renderer.createElement('mat-icon');
    const delIcon = this.renderer.createElement('mat-icon');

    this.renderer.appendChild(editIcon, this.renderer.createText('edit'));
    this.renderer.appendChild(delIcon, this.renderer.createText('delete'));

    this.renderer.appendChild(editBtn, editIcon);
    this.renderer.appendChild(delBtn, delIcon);

    this.renderer.addClass(editBtn, 'mat-icon-button');
    this.renderer.addClass(delBtn, 'mat-icon-button');

    this.renderer.appendChild(div, editBtn);
    this.renderer.appendChild(div, delBtn);

    this.renderer.listen(editBtn, 'click', () => this.onEditClicked(params.data));
    this.renderer.listen(delBtn, 'click', () => this.onDeleteClicked(params.data));

    return div;
  }

  dateFormatter(params: any): string {
    return new Date(params.value).toLocaleDateString();
  }

  booleanFormatter(params: any): string {
    return params.value ? 'Yes' : 'No';
  }

  showSnackBar(message: string): void {
    const config = new MatSnackBarConfig();
    config.duration = 3000;
    config.horizontalPosition = 'center';
    config.verticalPosition = 'bottom';
    this.snackBar.open(message, 'Close', config);
  }
}



import { Component, Input, OnChanges } from '@angular/core';
import { DashboardService } from '../dashboard.service';
import { Dashboardd } from '../dashboard.model';
import {
  ApexAxisChartSeries,
  ApexChart,
  ApexXAxis,
  ApexTitleSubtitle,
  ApexNonAxisChartSeries
} from 'ng-apexcharts';

@Component({
  selector: 'app-chart-viewer',
  templateUrl: './chart-viewer.component.html',
  styleUrls: ['./chart-viewer.component.css']
})
export class ChartViewerComponent implements OnChanges {
  @Input() dashboard: Dashboardd | null = null;

  chartOptions: {
    series?: ApexAxisChartSeries | ApexNonAxisChartSeries;
    chart?: ApexChart;
    xaxis?: ApexXAxis;
    title?: ApexTitleSubtitle;
    labels?: string[];
  } = {};

  constructor(private dashboardService: DashboardService) {}

  ngOnChanges(): void {
    if (this.dashboard) {
      console.log('Chart Viewer Input:', this.dashboard);
      this.loadChartData();
    }
  }

  loadChartData(): void {
    if (!this.dashboard) return;

    const { model, groupBy, aggregation, aggregationField, chartType } = this.dashboard;

    if (chartType === 'BarChart') {
      this.dashboardService.getBarChartData(model, groupBy, aggregation, aggregationField).subscribe(data => {
        this.chartOptions = {
          series: [{ name: 'Value', data: data.values }],
          chart: { type: 'bar', height: 350 },
          xaxis: { categories: data.categories },
          title: { text: String(this.dashboard?.name || 'Bar Chart') }
        };
      });
    } else if (chartType === 'PieChart') {
      this.dashboardService.getPieChartData(model, groupBy, aggregation, aggregationField).subscribe(data => {
        this.chartOptions = {
          series: data.values,
          chart: { type: 'pie', height: 350 },
          labels: data.categories,
          title: { text: String(this.dashboard?.name || 'Pie Chart') }
        };
      });
    }
  }
}
























mport { Component, Input, OnChanges } from '@angular/core';
import { ApexAxisChartSeries, ApexChart, ApexXAxis, ApexTitleSubtitle, ApexNonAxisChartSeries } from 'ng-apexcharts';
import { Dashboardd } from '../models/dashboard.model'; // adjust path if needed
import { DashboardService } from '../services/dashboard.service';

@Component({
  selector: 'app-chart-viewer',
  templateUrl: './chart-viewer.component.html',
  styleUrls: ['./chart-viewer.component.css']
})
export class ChartViewerComponent implements OnChanges {
  @Input() dashboard: Dashboardd | null = null;

  chartOptions: {
    series?: ApexAxisChartSeries | ApexNonAxisChartSeries;
    chart?: ApexChart;
    xaxis?: ApexXAxis;
    title?: ApexTitleSubtitle;
    labels?: string[];
  } = {};

  constructor(private dashboardService: DashboardService) {}

  ngOnChanges(): void {
    if (this.dashboard) {
      console.log('Chart Viewer Input:', this.dashboard);
      this.loadChartData();
    }
  }

  loadChartData(): void {
    if (this.dashboard?.chartType === 'BarChart') {
      this.dashboardService.getBarChartData(this.dashboard.id).subscribe(data => {
        this.chartOptions = {
          series: [{ name: 'Value', data: data.values }],
          chart: { type: 'bar', height: 350 },
          xaxis: { categories: data.categories },
          title: { text: this.dashboard?.name || 'Bar Chart' }
        };
      });
    } else if (this.dashboard?.chartType === 'PieChart') {
      this.dashboardService.getPieChartData(this.dashboard.id).subscribe(data => {
        this.chartOptions = {
          series: data.values,
          chart: { type: 'pie', height: 350 },
          labels: data.categories,
          title: { text: this.dashboard?.name || 'Pie Chart' }
        };
      });
    }
  }
}












<!-- src/app/components/chart-viewer/chart-viewer.component.html -->
<apx-chart
  *ngIf="chartOptions"
  [series]="chartOptions.series"
  [chart]="chartOptions.chart"
  [xaxis]="chartOptions.xaxis"
  [labels]="chartOptions.labels"
  [title]="chartOptions.title">
</apx-chart>

<!-- Chart Popup -->
<div *ngIf="selectedDashboardForView" class="popup-backdrop">
  <div class="popup-chart">
    <button class="close-btn" (click)="selectedDashboardForView = null">
      <mat-icon>close</mat-icon>
    </button>

    <app-chart-viewer [dashboard]="selectedDashboardForView"></app-chart-viewer>
  </div>
</div>


onDashboardNameClicked(dashboard: Dashboardd): void {
  this.selectedDashboardForView = dashboard;
}









// src/app/components/chart-viewer/chart-viewer.component.ts
import { Component, Input, OnChanges } from '@angular/core';
import { DashboardService } from '../services/dashboard.service';
import { Dashboardd } from '../models/dashboard.model';

@Component({
  selector: 'app-chart-viewer',
  templateUrl: './chart-viewer.component.html',
  styleUrls: ['./chart-viewer.component.scss']
})
export class ChartViewerComponent implements OnChanges {
  @Input() dashboard!: Dashboardd;
  chartOptions: any;

  constructor(private dashboardService: DashboardService) {}

  ngOnChanges(): void {
    if (this.dashboard) {
      const { model, groupBy, aggregation, aggregationField, chartType, name } = this.dashboard;

      if (chartType === 'BarChart') {
        this.dashboardService.getBarChartData(model, groupBy, aggregation, aggregationField)
          .subscribe(data => {
            this.chartOptions = {
              series: [{ name: aggregation, data: data.map(item => item.value) }],
              chart: { type: 'bar', height: 350 },
              xaxis: { categories: data.map(item => item.label) },
              title: { text: name }
            };
          });
      } else if (chartType === 'PieChart') {
        this.dashboardService.getPieChartData(model, groupBy, aggregation, aggregationField)
          .subscribe(data => {
            this.chartOptions = {
              series: data.map(item => item.value),
              chart: { type: 'pie', height: 350 },
              labels: data.map(item => item.label),
              title: { text: name }
            };
          });
      }
    }
  }
}






onDashboardNameClicked(dashboard: Dashboardd): void {
  this.selectedDashboardForView = dashboard;

  const { model, groupBy, aggregation, aggregationField, chartType, name } = dashboard;

  if (chartType === 'BarChart') {
    this.dashboardService.getBarChartData(model, groupBy, aggregation, aggregationField)
      .subscribe(data => {
        this.chartOptions = {
          series: [{ name: aggregation, data: data.map(item => item.value) }],
          chart: { type: 'bar', height: 350 },
          xaxis: { categories: data.map(item => item.label) },
          title: { text: name }
        };
      });
  } else if (chartType === 'PieChart') {
    this.dashboardService.getPieChartData(model, groupBy, aggregation, aggregationField)
      .subscribe(data => {
        this.chartOptions = {
          series: data.map(item => item.value),
          chart: { type: 'pie', height: 350 },
          labels: data.map(item => item.label),
          title: { text: name }
        };
      });
  }
}










controller
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
service 

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
        String query = String.format(
            "SELECT %s AS label, %s(%s) AS value FROM %s GROUP BY %s",
            groupBy, aggregation, aggregationField, model, groupBy
        );
        return jdbcTemplate.queryForList(query);
    }
}
















import {
  Component,
  EventEmitter,
  OnInit,
  Output,
  Renderer2,
  ElementRef,
  ChangeDetectorRef,
  CUSTOM_ELEMENTS_SCHEMA,
} from '@angular/core';
import { CommonModule } from '@angular/common';
import { MatIconModule } from '@angular/material/icon';
import { MatButtonModule } from '@angular/material/button';
import { MatSnackBar, MatSnackBarConfig } from '@angular/material/snack-bar';
import { AgGridModule, ColDef } from 'ag-grid-angular';
import { ApexChartModule } from 'ng-apexcharts';

import { Dashboardd } from './dashboard.model';
import { DashboardService } from './dashboard.service';
import { BlankDashboardComponent } from './blank-dashboard/blank-dashboard.component';
import { InteractiveDashboardComponent } from './interactive-dashboard/interactive-dashboard.component';

@Component({
  selector: 'app-manage-dashboard',
  standalone: true,
  imports: [
    CommonModule,
    MatIconModule,
    MatButtonModule,
    AgGridModule,
    ApexChartModule,
    BlankDashboardComponent,
    InteractiveDashboardComponent,
  ],
  templateUrl: './manage-dashboard.component.html',
  styleUrls: ['./manage-dashboard.component.css'],
  schemas: [CUSTOM_ELEMENTS_SCHEMA],
})
export class ManageDashboardComponent implements OnInit {
  tableData: Dashboardd[] = [];
  showBlankDashboard = false;
  showInteractiveDashboard = false;
  selectedBlankDashboard?: Dashboardd;
  selectedInteractiveDashboard?: Dashboardd;

  // Chart Popup
  selectedDashboardForView: Dashboardd | null = null;
  chartOptions: any;

  columnDefs: ColDef[] = [
    {
      headerName: 'Type',
      cellRenderer: this.typeIconCellRenderer.bind(this),
      width: 50,
    },
    {
      field: 'name',
      headerName: 'Name',
      cellRenderer: this.nameCellRenderer.bind(this),
    },
    { field: 'description', headerName: 'Description' },
    { field: 'createdBy', headerName: 'Created By' },
    { field: 'createdDate', headerName: 'Created Date', valueFormatter: this.dateFormatter },
    { field: 'modifiedBy', headerName: 'Modified By' },
    { field: 'modifiedDate', headerName: 'Modified Date', valueFormatter: this.dateFormatter },
    { field: 'isPublic', headerName: 'Public', valueFormatter: this.booleanFormatter },
    {
      headerName: 'Actions',
      cellRenderer: this.actionCellRenderer.bind(this),
    },
  ];

  defaultColDef: ColDef = {
    resizable: true,
    sortable: true,
    filter: true,
    flex: 1,
  };

  constructor(
    private dashboardService: DashboardService,
    private renderer: Renderer2,
    private elementRef: ElementRef,
    private changeDetectorRef: ChangeDetectorRef,
    private snackBar: MatSnackBar
  ) {}

  ngOnInit(): void {
    this.loadDashboards();
  }

  loadDashboards(): void {
    this.dashboardService.getDashboards().subscribe({
      next: (data: Dashboardd[]) => {
        this.tableData = data;
        this.changeDetectorRef.detectChanges();
      },
      error: (error) => {
        console.error('Error loading dashboards:', error);
      },
    });
  }

  onEditClicked(dashboard: Dashboardd): void {
    const isInteractive = !!(
      dashboard.model ||
      dashboard.groupBy ||
      dashboard.aggregation ||
      dashboard.aggregationField
    );

    if (isInteractive) {
      this.selectedInteractiveDashboard = dashboard;
      this.showInteractiveDashboard = true;
      this.showBlankDashboard = false;
    } else {
      this.selectedBlankDashboard = dashboard;
      this.showBlankDashboard = true;
      this.showInteractiveDashboard = false;
    }

    this.changeDetectorRef.detectChanges();
  }

  onDeleteClicked(data: any): void {
    this.dashboardService.deleteDashboard(data.id).subscribe({
      next: () => {
        this.showSnackBar('Dashboard deleted successfully');
        this.loadDashboards();
      },
      error: (error) => {
        console.error('Error deleting dashboard:', error);
      },
    });
  }

  onDashboardCreated(): void {
    this.showSnackBar('Dashboard created successfully');
    this.loadDashboards();
    this.onBackToManage();
  }

  onDashboardUpdated(): void {
    this.showSnackBar('Dashboard updated successfully');
    this.loadDashboards();
  }

  onBackToManage(): void {
    this.showBlankDashboard = false;
    this.showInteractiveDashboard = false;
    this.selectedBlankDashboard = undefined;
    this.selectedInteractiveDashboard = undefined;
    this.changeDetectorRef.detectChanges();
  }

  onClose(): void {
    this.onBackToManage();
    this.changeDetectorRef.detectChanges();
  }

  onGridReady(params: any): void {
    params.api.sizeColumnsToFit();
  }

  // 🟢 Chart Popup Handler
  loadChartPopup(dashboard: Dashboardd): void {
    this.selectedDashboardForView = dashboard;

    const { model, groupBy, aggregation, aggregationField, chartType } = dashboard;

    if (chartType === 'bar') {
      this.dashboardService.getBarChartData(model, groupBy, aggregation, aggregationField).subscribe({
        next: (data: any[]) => {
          this.chartOptions = {
            chart: {
              type: 'bar',
              height: 350,
            },
            series: [
              {
                name: 'Value',
                data: data.map((d: any) => d.value),
              },
            ],
            xaxis: {
              categories: data.map((d: any) => d.category),
            },
            title: {
              text: dashboard.name,
              align: 'center',
            },
          };
          this.changeDetectorRef.detectChanges();
        },
      });
    } else if (chartType === 'pie') {
      this.dashboardService.getPieChartData(model, groupBy, aggregation, aggregationField).subscribe({
        next: (data: any[]) => {
          this.chartOptions = {
            chart: {
              type: 'pie',
              height: 350,
            },
            series: data.map((d: any) => d.value),
            labels: data.map((d: any) => d.category),
            title: {
              text: dashboard.name,
              align: 'center',
            },
          };
          this.changeDetectorRef.detectChanges();
        },
      });
    }
  }

  nameCellRenderer(params: any): HTMLElement {
    const div = this.renderer.createElement('div');
    const link = this.renderer.createElement('a');
    this.renderer.setStyle(link, 'cursor', 'pointer');
    this.renderer.appendChild(link, this.renderer.createText(params.value));
    this.renderer.listen(link, 'click', () => this.loadChartPopup(params.data));
    this.renderer.appendChild(div, link);
    return div;
  }

  actionCellRenderer(params: any): HTMLElement {
    const div = this.renderer.createElement('div');
    const editBtn = this.renderer.createElement('button');
    const delBtn = this.renderer.createElement('button');
    const editIcon = this.renderer.createElement('mat-icon');
    const delIcon = this.renderer.createElement('mat-icon');

    this.renderer.appendChild(editIcon, this.renderer.createText('edit'));
    this.renderer.appendChild(delIcon, this.renderer.createText('delete'));

    this.renderer.appendChild(editBtn, editIcon);
    this.renderer.appendChild(delBtn, delIcon);
    this.renderer.addClass(editBtn, 'mat-icon-button');
    this.renderer.addClass(delBtn, 'mat-icon-button');

    this.renderer.appendChild(div, editBtn);
    this.renderer.appendChild(div, delBtn);

    this.renderer.listen(editBtn, 'click', () => this.onEditClicked(params.data));
    this.renderer.listen(delBtn, 'click', () => this.onDeleteClicked(params.data));

    return div;
  }

  typeIconCellRenderer(params: any): HTMLElement {
    const div = this.renderer.createElement('div');
    const icon = this.renderer.createElement('mat-icon');
    this.renderer.appendChild(icon, this.renderer.createText(params.data.type === 'blank' ? 'speed' : 'analytics'));
    this.renderer.appendChild(div, icon);
    return div;
  }

  dateFormatter(params: any): string {
    return new Date(params.value).toLocaleDateString();
  }

  booleanFormatter(params: any): string {
    return params.value ? 'Yes' : 'No';
  }

  showSnackBar(message: string): void {
    const config = new MatSnackBarConfig();
    config.duration = 3000;
    config.horizontalPosition = 'center';
    config.verticalPosition = 'bottom';
    this.snackBar.open(message, 'Close', config);
  }
}



<h1>All Dashboards</h1>

<ag-grid-angular
  class="ag-theme-balham"
  style="width: 100%; height: 500px;"
  [rowData]="tableData"
  [columnDefs]="columnDefs"
  [defaultColDef]="defaultColDef"
  (gridReady)="onGridReady($event)">
</ag-grid-angular>

<!-- Chart Popup -->
<div *ngIf="selectedDashboardForView" class="popup-backdrop">
  <div class="popup-chart">
    <button class="close-btn" (click)="selectedDashboardForView = null">
      <mat-icon>close</mat-icon>
    </button>
    <apx-chart
      *ngIf="chartOptions"
      [series]="chartOptions.series"
      [chart]="chartOptions.chart"
      [xaxis]="chartOptions.xaxis"
      [labels]="chartOptions.labels"
      [title]="chartOptions.title"
    ></apx-chart>
  </div>
</div>























import { Component, Input, OnChanges, SimpleChanges, Output, EventEmitter } from '@angular/core';
import { CommonModule } from '@angular/common';
import { NgApexchartsModule } from 'ng-apexcharts';
import { DashboardService } from '../dashboard.service';
import { ApexChart, ApexAxisChartSeries, ApexXAxis, ApexNonAxisChartSeries } from 'ng-apexcharts';

@Component({
  selector: 'app-chart-viewer',
  standalone: true,
  imports: [CommonModule, NgApexchartsModule],
  templateUrl: './chart-viewer.component.html',
  styleUrls: ['./chart-viewer.component.css']
})
export class ChartViewerComponent implements OnChanges {
  @Input() dashboard: any;
  @Output() close = new EventEmitter<void>();

  barChartOptions: {
    series: ApexAxisChartSeries;
    chart: ApexChart;
    xaxis: ApexXAxis;
  } = {
    series: [],
    chart: { type: 'bar', height: 350 },
    xaxis: { categories: [] }
  };

  pieChartOptions: {
    series: ApexNonAxisChartSeries;
    chart: ApexChart;
    labels: string[];
  } = {
    series: [],
    chart: { type: 'pie', height: 350 },
    labels: []
  };

  constructor(private dashboardService: DashboardService) {}

  ngOnChanges(changes: SimpleChanges): void {
    if (changes['dashboard'] && this.dashboard) {
      this.loadChartData();
    }
  }

  loadChartData(): void {
    const { model, groupBy, aggregation, aggregationField, chartType } = this.dashboard;

    if (chartType === 'bar') {
      this.dashboardService.getBarChartData(model, groupBy, aggregation, aggregationField).subscribe(data => {
        this.barChartOptions = {
          series: [{ name: aggregation, data: data.values }],
          chart: { type: 'bar', height: 350 },
          xaxis: { categories: data.labels }
        };
      });
    } else if (chartType === 'pie') {
      this.dashboardService.getPieChartData(model, groupBy, aggregation, aggregationField).subscribe(data => {
        this.pieChartOptions = {
          series: data.values,
          chart: { type: 'pie', height: 350 },
          labels: data.labels
        };
      });
    }
  }

  closeViewer(): void {
    this.close.emit();
  }
}


<div class="chart-popup-backdrop">
  <div class="chart-popup">
    <h2>{{ dashboard?.title }} ({{ dashboard?.chartType | titlecase }} Chart)</h2>

    <apx-chart
      *ngIf="dashboard?.chartType === 'bar'"
      [series]="barChartOptions.series"
      [chart]="barChartOptions.chart"
      [xaxis]="barChartOptions.xaxis">
    </apx-chart>

    <apx-chart
      *ngIf="dashboard?.chartType === 'pie'"
      [series]="pieChartOptions.series"
      [chart]="pieChartOptions.chart"
      [labels]="pieChartOptions.labels">
    </apx-chart>

    <button mat-button color="warn" (click)="closeViewer()">Close</button>
  </div>
</div>


<app-chart-viewer
  *ngIf="selectedDashboardForView"
  [dashboard]="selectedDashboardForView"
  (close)="selectedDashboardForView = null">
</app-chart-viewer>

.chart-popup-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100dvw;
  height: 100dvh;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.chart-popup {
  background-color: white;
  padding: 20px;
  border-radius: 10px;
  width: 90%;
  max-width: 800px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
}






















Step 7: Show Chart Viewer in HTML

In manage-dashboard.component.html, add this:

<app-chart-viewer
  *ngIf="selectedDashboardForView"
  [dashboard]="selectedDashboardForView"
></app-chart-viewer>


---

🔹 Step 8: Update Your DashboardService

Ensure these methods are in your service and return Observable<any[]>:

getBarchartData(dashboard: Dashboardd): Observable<any[]> {
  return this.http.post<any[]>(`/api/bar-data`, dashboard);
}

getPieChartData(dashboard: Dashboardd): Observable<any[]> {
  return this.http.post<any[]>(`/api/pie-data`, dashboard);
}

Make sure your backend accepts the dashboard input and returns:

[
  { label: "Category A", value: 10 },
  { label: "Category B", value: 20 }
]






















Step 3: Create <app-chart-viewer> Component

Run this command to generate it:

ng generate component chart-viewer --standalone


---

🔹 Step 4: Implement chart-viewer.component.ts

import { Component, Input, OnChanges, SimpleChanges } from '@angular/core';
import { DashboardService } from '../dashboard.service';
import { Dashboardd } from '../models/dashboard.model';
import {
  ApexNonAxisChartSeries,
  ApexChart,
  ApexXAxis,
  ApexDataLabels,
  ApexTitleSubtitle,
  ApexResponsive
} from 'ng-apexcharts';

export type ChartOptions = {
  series: ApexNonAxisChartSeries;
  chart: ApexChart;
  labels?: any;
  xaxis?: ApexXAxis;
  dataLabels?: ApexDataLabels;
  title?: ApexTitleSubtitle;
  responsive?: ApexResponsive[];
};

@Component({
  selector: 'app-chart-viewer',
  templateUrl: './chart-viewer.component.html',
  standalone: true,
  styleUrls: ['./chart-viewer.component.css'],
  imports: [],
})
export class ChartViewerComponent implements OnChanges {
  @Input() dashboard!: Dashboardd;
  chartOptions!: Partial<ChartOptions>;
  chartType: 'bar' | 'pie' = 'bar';

  constructor(private dashboardService: DashboardService) {}

  ngOnChanges(changes: SimpleChanges): void {
    if (changes['dashboard'] && this.dashboard) {
      this.loadChart();
    }
  }

  loadChart(): void {
    if (this.dashboard.chartType === 'PieChart') {
      this.chartType = 'pie';
      this.dashboardService.getPieChartData(this.dashboard).subscribe((data) => {
        this.chartOptions = {
          series: data.map((item: any) => item.value),
          chart: {
            type: 'pie',
            width: 380,
          },
          labels: data.map((item: any) => item.label),
        };
      });
    } else {
      this.chartType = 'bar';
      this.dashboardService.getBarchartData(this.dashboard).subscribe((data) => {
        this.chartOptions = {
          series: [{
            name: 'Value',
            data: data.map((item: any) => item.value),
          }],
          chart: {
            type: 'bar',
            height: 350,
          },
          xaxis: {
            categories: data.map((item: any) => item.label),
          },
        };
      });
    }
  }
}


---

🔹 Step 5: chart-viewer.component.html

<div class="chart-modal">
  <div class="chart-popup">
    <button class="close-btn" (click)="dashboard = null">X</button>

    <apx-chart
      *ngIf="chartOptions"
      [series]="chartOptions.series"
      [chart]="chartOptions.chart"
      [labels]="chartOptions.labels"
      [xaxis]="chartOptions.xaxis"
      [dataLabels]="chartOptions.dataLabels"
      [title]="chartOptions.title"
      [responsive]="chartOptions.responsive"
    ></apx-chart>
  </div>
</div>


---

🔹 Step 6: chart-viewer.component.css

.chart-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100dvw;
  height: 100dvh;
  background: rgba(0, 0, 0, 0.6);
  z-index: 1000;
  display: flex;
  align-items: center;
  justify-content: center;
}

.chart-popup {
  background: white;
  padding: 1rem;
  border-radius: 8px;
  width: 80%;
  max-width: 600px;
  position: relative;
}

.close-btn {
  position: absolute;
  top: 10px;
  right: 15px;
  font-size: 18px;
  background: none;
  border: none;
  cursor: pointer;
}
























<div class="manage-dashboard-container" style="width: 90vw; overflow-x: auto;">
  <!-- Header -->
  <div class="header">
    <h1 class="title">Manage Dashboard</h1>
    <button class="close-btn" (click)="onClose()">
      <mat-icon>close</mat-icon>
    </button>
  </div>

  <!-- New Dashboard Buttons -->
  <div class="new-dashboard-section">
    <div class="dashboard-types">
      <div class="dashboard-card" (click)="toggleBlankDashboard()">
        <mat-icon class="dashboard-icon">speed</mat-icon>
        <div class="dashboard-name">Blank Dashboard</div>
      </div>
      <div class="dashboard-card" (click)="toggleInteractiveDashboard()">
        <mat-icon class="dashboard-icon">analytics</mat-icon>
        <div class="dashboard-name">Interactive Dashboard</div>
      </div>
    </div>
  </div>

  <!-- All Dashboards Table -->
  <div class="all-dashboards-section">
    <h2 class="section-title">All Dashboards</h2>
    <div class="table-container">
      <ag-grid-angular
        class="ag-theme-balham"
        style="width: 100%; height: 500px;"
        [gridOptions]="gridOptions"
        [rowData]="tableData"
        (gridReady)="onGridReady($event)">
      </ag-grid-angular>
    </div>
  </div>

  <!-- Chart Viewer Popup -->
  <div *ngIf="selectedDashboardForView" class="chart-popup-backdrop">
    <div class="chart-popup">
      <h2>{{ selectedDashboardForView.name }} ({{ selectedDashboardForView.chartType | titlecase }} Chart)</h2>

      <apx-chart
        *ngIf="selectedDashboardForView.chartType === 'bar'"
        [series]="barChartOptions.series"
        [chart]="barChartOptions.chart"
        [xaxis]="barChartOptions.xaxis">
      </apx-chart>

      <apx-chart
        *ngIf="selectedDashboardForView.chartType === 'pie'"
        [series]="pieChartOptions.series"
        [chart]="pieChartOptions.chart"
        [labels]="pieChartOptions.labels">
      </apx-chart>

      <button mat-button color="warn" (click)="closeChartViewer()">Close</button>
    </div>
  </div>

  <!-- Blank Dashboard Modal -->
  <app-blank-dashboard
    *ngIf="showBlankDashboard"
    [editData]="selectedBlankDashboard"
    (dashboardClose)="onClose()"
    (dashboardCreated)="onDashboardCreated()"
    (dashboardUpdated)="onDashboardUpdated()">
  </app-blank-dashboard>

  <!-- Interactive Dashboard Modal -->
  <app-interactive-dashboard
    *ngIf="showInteractiveDashboard"
    [editData]="selectedInteractiveDashboard"
    (dashboardClose)="onClose()"
    (dashboardCreated)="onDashboardCreated()"
    (dashboardUpdated)="onDashboardUpdated()">
  </app-interactive-dashboard>
</div>

.chart-popup-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 999;
}

.chart-popup {
  background: white;
  padding: 20px;
  border-radius: 12px;
  max-width: 700px;
  width: 90%;
}




















import {
  Component,
  EventEmitter,
  OnInit,
  Output,
  Renderer2,
  ElementRef,
  ChangeDetectorRef,
  CUSTOM_ELEMENTS_SCHEMA,
} from '@angular/core';
import { CommonModule } from '@angular/common';
import { NgIf } from '@angular/common';
import { MatIconModule } from '@angular/material/icon';
import { MatButtonModule } from '@angular/material/button';
import { MatSnackBar, MatSnackBarConfig } from '@angular/material/snack-bar';
import { ApexChartModule } from 'ng-apexcharts';
import { AgGridModule } from 'ag-grid-angular';
import { ColDef } from 'ag-grid-community';
import { DashboardService } from './dashboard.service';
import { Dashboardd } from './dashboard.model';
import { BlankDashboardComponent } from './blank-dashboard/blank-dashboard.component';
import { InteractiveDashboardComponent } from './interactive-dashboard/interactive-dashboard.component';

@Component({
  selector: 'app-manage-dashboard',
  standalone: true,
  templateUrl: './manage-dashboard.component.html',
  styleUrls: ['./manage-dashboard.component.css'],
  imports: [
    CommonModule,
    NgIf,
    MatIconModule,
    MatButtonModule,
    AgGridModule,
    BlankDashboardComponent,
    InteractiveDashboardComponent,
    ApexChartModule,
  ],
  schemas: [CUSTOM_ELEMENTS_SCHEMA],
})
export class ManageDashboardComponent implements OnInit {
  tableData: Dashboardd[] = [];
  selectedDashboard?: Dashboardd;
  showBlankDashboard = false;
  showInteractiveDashboard = false;
  selectedBlankDashboard?: Dashboardd;
  selectedInteractiveDashboard?: Dashboardd;

  // Apex chart variables
  selectedDashboardForView?: Dashboardd;
  barChartOptions: any = {};
  pieChartOptions: any = {};

  @Output() closeDashboard = new EventEmitter<void>();
  @Output() selectDashboard = new EventEmitter<string>();

  columnDefs: ColDef[] = [
    {
      headerName: 'Type Icon',
      cellRenderer: this.typeIconCellRenderer.bind(this),
      width: 50,
    },
    {
      field: 'name',
      headerName: 'Name',
      cellRenderer: this.nameCellRenderer.bind(this),
    },
    { field: 'description', headerName: 'Description' },
    { field: 'createdBy', headerName: 'Created By' },
    {
      field: 'createdDate',
      headerName: 'Created Date',
      valueFormatter: this.dateFormatter,
    },
    {
      field: 'modifiedBy',
      headerName: 'Modified By',
    },
    {
      field: 'modifiedDate',
      headerName: 'Modified Date',
      valueFormatter: this.dateFormatter,
    },
    {
      field: 'isPublic',
      headerName: 'Public',
      valueFormatter: this.booleanFormatter,
    },
    {
      headerName: 'Actions',
      cellRenderer: this.actionCellRenderer.bind(this),
    },
  ];

  defaultColDef: ColDef = {
    resizable: true,
    sortable: true,
    filter: true,
    flex: 1,
  };

  gridOptions: any = {
    columnDefs: this.columnDefs,
    defaultColDef: this.defaultColDef,
    rowData: this.tableData,
    onGridReady: this.onGridReady.bind(this),
    rowSelection: 'single',
    pagination: true,
    paginationPageSize: 10,
    suppressRowClickSelection: true,
    animateRows: true,
  };

  constructor(
    private dashboardService: DashboardService,
    private renderer: Renderer2,
    private elementRef: ElementRef,
    private changeDetectorRef: ChangeDetectorRef,
    private snackBar: MatSnackBar
  ) {}

  ngOnInit(): void {
    this.loadDashboards();
  }

  loadDashboards(): void {
    this.dashboardService.getDashboards().subscribe({
      next: (data: Dashboardd[]) => {
        this.tableData = data;
        this.gridOptions.api.setRowData(this.tableData);
        this.changeDetectorRef.detectChanges();
      },
      error: (error) => {
        console.error('Error loading dashboards:', error);
      },
    });
  }

  dateFormatter(params: any): string {
    return new Date(params.value).toLocaleDateString();
  }

  booleanFormatter(params: any): string {
    return params.value ? 'Yes' : 'No';
  }

  onEditClicked(dashboard: Dashboardd): void {
    const isInteractive =
      !!(dashboard.model || dashboard.groupBy || dashboard.aggregation || dashboard.aggregationField);

    if (isInteractive) {
      this.selectedInteractiveDashboard = dashboard;
      this.showInteractiveDashboard = true;
      this.showBlankDashboard = false;
    } else {
      this.selectedBlankDashboard = dashboard;
      this.showBlankDashboard = true;
      this.showInteractiveDashboard = false;
    }

    this.changeDetectorRef.detectChanges();
  }

  onDeleteClicked(data: any): void {
    this.dashboardService.deleteDashboard(data.id).subscribe({
      next: () => {
        this.showSnackBar('Dashboard deleted successfully');
        this.loadDashboards();
      },
      error: (err) => console.error('Error deleting dashboard:', err),
    });
  }

  onDashboardCreated(): void {
    this.showSnackBar('Dashboard created successfully');
    this.loadDashboards();
    this.onBackToManage();
  }

  onDashboardUpdated(): void {
    this.showSnackBar('Dashboard updated successfully');
    this.loadDashboards();
  }

  onBackToManage(): void {
    this.showBlankDashboard = false;
    this.showInteractiveDashboard = false;
    this.selectedBlankDashboard = undefined;
    this.selectedInteractiveDashboard = undefined;
    this.selectedDashboardForView = undefined;
    this.changeDetectorRef.detectChanges();
  }

  onClose(): void {
    this.onBackToManage();
    this.closeDashboard.emit();
  }

  actionCellRenderer(params: any): HTMLElement {
    const div = this.renderer.createElement('div');
    const editButton = this.renderer.createElement('button');
    const deleteButton = this.renderer.createElement('button');
    this.renderer.addClass(editButton, 'mat-icon-button');
    this.renderer.addClass(deleteButton, 'mat-icon-button');

    const editIcon = this.renderer.createElement('mat-icon');
    const deleteIcon = this.renderer.createElement('mat-icon');
    this.renderer.appendChild(editIcon, this.renderer.createText('edit'));
    this.renderer.appendChild(deleteIcon, this.renderer.createText('delete'));

    this.renderer.appendChild(editButton, editIcon);
    this.renderer.appendChild(deleteButton, deleteIcon);
    this.renderer.appendChild(div, editButton);
    this.renderer.appendChild(div, deleteButton);

    this.renderer.listen(editButton, 'click', () => {
      this.onEditClicked(params.data);
    });
    this.renderer.listen(deleteButton, 'click', () => {
      this.onDeleteClicked(params.data);
    });

    return div;
  }

  typeIconCellRenderer(params: any): HTMLElement {
    const div = this.renderer.createElement('div');
    const icon = this.renderer.createElement('span');
    this.renderer.addClass(icon, 'material-icons');
    const iconType = params.data.type === 'blank' ? 'speed' : 'analytics';
    this.renderer.appendChild(icon, this.renderer.createText(iconType));
    this.renderer.appendChild(div, icon);
    return div;
  }

  nameCellRenderer(params: any): HTMLElement {
    const div = this.renderer.createElement('div');
    const link = this.renderer.createElement('a');
    this.renderer.setStyle(link, 'cursor', 'pointer');
    this.renderer.setStyle(link, 'color', '#007bff');
    this.renderer.setStyle(link, 'textDecoration', 'underline');
    this.renderer.appendChild(link, this.renderer.createText(params.value));
    this.renderer.listen(link, 'click', () => {
      this.onDashboardNameClicked(params.data);
    });
    this.renderer.appendChild(div, link);
    return div;
  }

  onDashboardNameClicked(dashboard: Dashboardd): void {
    this.selectedDashboardForView = dashboard;

    if (dashboard.chartType === 'bar') {
      this.dashboardService
        .getBarChartData(dashboard.model, dashboard.groupBy, dashboard.aggregation, dashboard.aggregationField)
        .subscribe({
          next: (data) => {
            this.barChartOptions = {
              chart: {
                type: 'bar',
              },
              series: [
                {
                  name: 'Value',
                  data: data.map((item: any) => item.value),
                },
              ],
              xaxis: {
                categories: data.map((item: any) => item.category),
              },
            };
            this.changeDetectorRef.detectChanges();
          },
        });
    } else if (dashboard.chartType === 'pie') {
      this.dashboardService
        .getPieChartData(dashboard.model, dashboard.groupBy, dashboard.aggregation, dashboard.aggregationField)
        .subscribe({
          next: (data) => {
            this.pieChartOptions = {
              chart: {
                type: 'pie',
              },
              labels: data.map((item: any) => item.category),
              series: data.map((item: any) => item.value),
            };
            this.changeDetectorRef.detectChanges();
          },
        });
    }
  }

  closeChartViewer(): void {
    this.selectedDashboardForView = undefined;
    this.changeDetectorRef.detectChanges();
  }

  showSnackBar(message: string): void {
    const config = new MatSnackBarConfig();
    config.horizontalPosition = 'center';
    config.verticalPosition = 'bottom';
    config.panelClass = ['custom-snackbar'];
    config.duration = 3000;
    this.snackBar.open(message, 'close', config);
  }

  onGridReady(params: any): void {
    params.api.sizeColumnsToFit();
  }
}


<!-- Chart Viewer Popup -->
<div *ngIf="selectedDashboardForView" class="chart-popup-backdrop">
  <div class="chart-popup">
    <h2>{{ selectedDashboardForView.name }} ({{ selectedDashboardForView.chartType | titlecase }} Chart)</h2>

    <apx-chart
      *ngIf="selectedDashboardForView.chartType === 'bar'"
      [series]="barChartOptions.series"
      [chart]="barChartOptions.chart"
      [xaxis]="barChartOptions.xaxis"
    ></apx-chart>

    <apx-chart
      *ngIf="selectedDashboardForView.chartType === 'pie'"
      [series]="pieChartOptions.series"
      [chart]="pieChartOptions.chart"
      [labels]="pieChartOptions.labels"
    ></apx-chart>

    <button mat-button color="warn" (click)="closeChartViewer()">Close</button>
  </div>
</div>












  




















html
<div class="manage-dashboard-container">
  <h1 class="title">Manage Dashboards</h1>

  <div class="table-container">
    <ag-grid-angular
      class="ag-theme-balham"
      style="width: 100%; height: 500px;"
      [gridOptions]="gridOptions"
      [rowData]="tableData"
    ></ag-grid-angular>
  </div>

  <!-- Chart Popup -->
  <div *ngIf="showChartPopup" class="chart-popup">
    <div class="chart-header">
      <h2>{{ selectedDashboard?.name }} - {{ selectedDashboard?.chartType }} Chart</h2>
      <button mat-icon-button (click)="closeChartPopup()">
        <mat-icon>close</mat-icon>
      </button>
    </div>

    <div *ngIf="selectedDashboard?.chartType === 'bar'">
      <ag-charts-angular [options]="barChartOptions" style="height: 400px;"></ag-charts-angular>
    </div>

    <div *ngIf="selectedDashboard?.chartType === 'pie'">
      <ag-charts-angular [options]="pieChartOptions" style="height: 400px;"></ag-charts-angular>
    </div>
  </div>

  <!-- Blank Dashboard Modal -->
  <app-blank-dashboard
    *ngIf="showBlankDashboard"
    [isEditMode]="true"
    [editData]="selectedBlankDashboard"
    (dashboardClose)="closeBlankDashboard()"
  ></app-blank-dashboard>

  <!-- Interactive Dashboard Modal -->
  <app-interactive-dashboard
    *ngIf="showInteractiveDashboard"
    [isEditMode]="true"
    [editData]="selectedInteractiveDashboard"
    (dashboardClose)="closeInteractiveDashboard()"
  ></app-interactive-dashboard>
</div>

css
.manage-dashboard-container {
  padding: 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}
,












































typeIconCellRenderer(params: any): HTMLElement {
  const div = this.renderer.createElement('div');
  const iconButton = this.renderer.createElement('button');
  const icon = this.renderer.createElement('span');

  this.renderer.addClass(iconButton, 'mat-icon-button');
  this.renderer.addClass(icon, 'material-icons'); // <-- Use material-icons class

  const iconType = params.data.type === 'blank' ? 'speed' : 'analytics';
  this.renderer.appendChild(icon, this.renderer.createText(iconType));

  this.renderer.appendChild(iconButton, icon);
  this.renderer.appendChild(div, iconButton);
  return div;
}






const uid = sessionStorage.getItem('uid') || 'unknown';  // fallback if uid not found

const dashboardPayload: Dashboardd = {
  name: this.formData.name,
  description: this.formData.description,
  isPublic: this.formData.isPublic,
  chartType: this.selectedChart,
  model: this.model,
  groupBy: this.groupBy,
  aggregation: this.aggregation,
  aggregationField: this.aggregationField,
  createdBy: this.editData?.createdBy ?? uid,
  createdDate: this.editData?.createdDate ?? new Date().toISOString(),
  modifiedBy: this.isEditMode ? uid : undefined,
  modifiedDate: new Date().toISOString(),
  type: 'blank',
};














interactive dashboard comp ts
import { Component, Input, Output, EventEmitter, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';
import { AgChartsModule } from 'ag-charts-angular';

@Component({
  selector: 'app-interactive-dashboard',
  standalone: true,
  imports: [CommonModule, FormsModule, AgChartsModule],
  templateUrl: './interactive-dashboard.component.html',
  styleUrls: ['./interactive-dashboard.component.css']
})
export class InteractiveDashboardComponent implements OnInit {
  @Input() editData: any;
  @Output() dashboardClose = new EventEmitter<void>();

  showModal = true;
  isFullscreen = false;
  isEditMode = false;
  currentStep = 0;
  steps = ['Initial', 'Content', 'Review'];

  formData = {
    name: '',
    description: '',
    isPublic: true
  };

  model: string = '';
  groupBy: string = '';
  aggregation: string = '';
  aggregationField: string = '';
  columnField: string = '';

  availableTemplates = ['BarChart', 'PieChart'];
  selectedTemplates: string[] = [];

  ngOnInit(): void {
    if (this.editData) {
      this.isEditMode = true;
      this.formData.name = this.editData.name;
      this.formData.description = this.editData.description;
      this.formData.isPublic = this.editData.isPublic;
      this.model = this.editData.model;
      this.groupBy = this.editData.groupBy;
      this.aggregation = this.editData.aggregation;
      this.aggregationField = this.editData.aggregationField;
      this.columnField = this.editData.columnField;
      this.selectedTemplates = this.editData.selectedTemplates || [];
    }
  }

  nextStep(): void {
    if (this.currentStep < this.steps.length - 1) {
      this.currentStep++;
    }
  }

  prevStep(): void {
    if (this.currentStep > 0) {
      this.currentStep--;
    }
  }

  toggleTemplate(template: string): void {
    if (!this.selectedTemplates.includes(template)) {
      this.selectedTemplates.push(template);
    }
  }

  removeTemplate(template: string): void {
    this.selectedTemplates = this.selectedTemplates.filter(t => t !== template);
  }

  toggleFullscreen(): void {
    this.isFullscreen = !this.isFullscreen;
  }

  submitDashboard(): void {
    const dashboardPayload = {
      name: this.formData.name,
      description: this.formData.description,
      isPublic: this.formData.isPublic,
      model: this.model,
      groupBy: this.groupBy,
      aggregation: this.aggregation,
      aggregationField: this.aggregationField,
      columnField: this.columnField,
      selectedTemplates: this.selectedTemplates,
      createdBy: this.editData?.createdBy || 'h59606',
      createdDate: this.editData?.createdDate || new Date().toISOString(),
      modifiedBy: 'h59606',
      modifiedDate: new Date().toISOString(),
    };

    // Make HTTP POST or PUT call here
    this.dashboardClose.emit();
  }

  closeModal(): void {
    this.dashboardClose.emit();
  }
}

interactive dashboard comp html
<div class="modal" [class.fullscreen]="isFullscreen">
  <div class="modal-header">
    <h2>{{ isEditMode ? 'Edit' : 'Create' }} Interactive Dashboard</h2>
    <button (click)="toggleFullscreen()">🗖</button>
    <button (click)="closeModal()">✖</button>
  </div>

  <div class="steps">
    <button *ngFor="let step of steps; let i = index"
            [class.active]="i === currentStep"
            (click)="currentStep = i">
      {{ step }}
    </button>
  </div>

  <div class="step-content" *ngIf="currentStep === 0">
    <label>Name:</label>
    <input [(ngModel)]="formData.name" type="text" />
    <label>Description:</label>
    <textarea [(ngModel)]="formData.description"></textarea>
    <label>
      <input type="checkbox" [(ngModel)]="formData.isPublic" />
      Public
    </label>
  </div>

  <div class="step-content" *ngIf="currentStep === 1">
    <label>Model:</label>
    <select [(ngModel)]="model">
      <option>Germany_Nostro</option>
      <option>Monaco_Nostro</option>
      <option>Switzerland_Nostro</option>
    </select>

    <label>Group By:</label>
    <select [(ngModel)]="groupBy">
      <option *ngFor="let item of ['Account', 'AccountNumber', 'Balance', 'Balance Pool', 'ClosingAvlBal', 'ClosingAvlBalCur', 'ClosingAvlBalDate', 'ClosingBalCur']">{{ item }}</option>
    </select>

    <label>Aggregation:</label>
    <select [(ngModel)]="aggregation">
      <option>Count</option>
      <option>Sum</option>
      <option>None</option>
    </select>

    <label>Aggregation Field:</label>
    <select [(ngModel)]="aggregationField">
      <option *ngFor="let item of ['Balance', 'ClosingAvlBal', 'ClosingBalance', 'Eq Balance', 'Opening Balance', 'System Balance', 'UserNum1', 'UserNum2']">{{ item }}</option>
    </select>

    <label>Column Field:</label>
    <select [(ngModel)]="columnField">
      <option *ngFor="let item of ['Account', 'AccountNumber', 'Balance', 'Balance Pool', 'ClosingAvlBal', 'ClosingAvlBalCur', 'ClosingAvlBalDate', 'ClosingBalCur']">{{ item }}</option>
    </select>

    <div class="template-section">
      <h4>Available Templates</h4>
      <button *ngFor="let template of availableTemplates" (click)="toggleTemplate(template)">
        {{ template }}
      </button>
      <div>
        <h4>Selected Templates</h4>
        <div *ngFor="let t of selectedTemplates">
          {{ t }} <button (click)="removeTemplate(t)">-</button>
        </div>
      </div>
    </div>
  </div>

  <div class="step-content" *ngIf="currentStep === 2">
    <p><strong>Name:</strong> {{ formData.name }}</p>
    <p><strong>Description:</strong> {{ formData.description }}</p>
    <p><strong>Visibility:</strong> {{ formData.isPublic ? 'Public' : 'Private' }}</p>
    <p><strong>Model:</strong> {{ model }}</p>
    <p><strong>GroupBy:</strong> {{ groupBy }}</p>
    <p><strong>Aggregation:</strong> {{ aggregation }}</p>
    <p><strong>Aggregation Field:</strong> {{ aggregationField }}</p>
    <p><strong>Column Field:</strong> {{ columnField }}</p>
    <p><strong>Templates:</strong> {{ selectedTemplates.join(', ') }}</p>
  </div>

  <div class="modal-footer">
    <button (click)="prevStep()" [disabled]="currentStep === 0">Previous</button>
    <button *ngIf="currentStep < steps.length - 1" (click)="nextStep()">Next</button>
    <button *ngIf="currentStep === steps.length - 1" (click)="submitDashboard()">Submit</button>
  </div>
</div>

css
.modal {
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.2);
  padding: 20px;
  max-width: 800px;
  margin: 2rem auto;
}
.fullscreen {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  margin: 0;
  width: 100%;
  height: 100%;
  z-index: 1000;
}
.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.steps {
  margin-top: 1rem;
}
.steps button {
  margin-right: 8px;
  padding: 6px 12px;
}
.steps button.active {
  background-color: #007bff;
  color: white;
}
.step-content {
  margin-top: 1rem;
}
.template-section {
  margin-top: 1rem;
}
.modal-footer {
  margin-top: 1.5rem;
  display: flex;
  justify-content: space-between;
}


manage dashboard comp html 
<!-- manage-dashboard.component.html --><div class="dashboard-table">
  <table>
    <thead>
      <tr>
        <th>Name</th>
        <th>Description</th>
        <th>Created By</th>
        <th>Type</th>
        <th>Actions</th>
      </tr>
    </thead>
    <tbody>
      <tr *ngFor="let dashboard of dashboards">
        <td>{{ dashboard.name }}</td>
        <td>{{ dashboard.description }}</td>
        <td>{{ dashboard.createdBy }}</td>
        <td>
          <span *ngIf="isInteractiveDashboard(dashboard)" class="interactive-badge">Interactive</span>
          <span *ngIf="!isInteractiveDashboard(dashboard)" class="blank-badge">Blank</span>
        </td>
        <td>
          <button (click)="editDashboard(dashboard)">✏️</button>
          <button (click)="deleteDashboard(dashboard.id)">🗑️</button>
        </td>
      </tr>
    </tbody>
  </table>
</div><!-- Modals --><app-blank-dashboard *ngIf="showBlankModal" [editData]="selectedBlankDashboard" (dashboardClose)="handleCloseBlank()"> </app-blank-dashboard>

<app-interactive-dashboard *ngIf="showInteractiveModal" [editData]="selectedInteractiveDashboard" (dashboardClose)="handleCloseInteractive()"> </app-interactive-dashboard>

ts

dashboards: DashboardDTO[] = [];
showBlankModal = false;
showInteractiveModal = false;
selectedBlankDashboard?: DashboardDTO;
selectedInteractiveDashboard?: DashboardDTO;

ngOnInit() {
  this.dashboardService.getAllDashboards().subscribe(data => {
    this.dashboards = data;
  });
}

isInteractiveDashboard(d: DashboardDTO): boolean {
  return !!(d.model && d.groupBy && d.aggregation); // Basic check for interactive fields




<div class="table-container">
  <table class="dashboards-table">
    <thead>
      <tr>
        <th *ngFor="let header of tableHeaders">{{ header.label }}</th>
      </tr>
    </thead>
    <tbody>
      <tr *ngIf="tableData.length === 0" class="no-data-row">
        <td [attr.colspan]="tableHeaders.length" class="no-data-cell">No Rows To Show</td>
      </tr>
      <tr *ngFor="let row of tableData">
        <td *ngFor="let header of tableHeaders">
          <ng-container [ngSwitch]="header.key">
            <span *ngSwitchCase="'createdDate'">{{ row.createdDate | date: 'short' }}</span>
            <span *ngSwitchCase="'modifiedDate'">{{ row.modifiedDate | date: 'short' }}</span>
            <span *ngSwitchCase="'public'">{{ row.isPublic ? 'Yes' : 'No' }}</span>
            <span *ngSwitchCase="'action'">
              <button mat-icon-button color="primary" (click)="onEditDashboard(row)">
                <mat-icon>edit</mat-icon>
              </button>
              <button mat-icon-button color="warn" (click)="onDeleteDashboard(row.id)">
                <mat-icon>delete</mat-icon>
              </button>
            </span>
            <span *ngSwitchDefault>{{ row[header.key] }}</span>
          </ng-container>
        </td>
      </tr>
    </tbody>
  </table>
</div>

mangage dashboard comp.ts
import { Component, OnInit, Output, EventEmitter } from '@angular/core';
import { DashboardService } from '../services/dashboard.service';
import { Dashboardd } from '../models/dashboard.model'; // adjust path as needed

@Component({
  selector: 'app-manage-dashboard',
  templateUrl: './manage-dashboard.component.html',
  styleUrls: ['./manage-dashboard.component.css']
})
export class ManageDashboardComponent implements OnInit {
  dashboards: Dashboardd[] = [];
  tableData: Dashboardd[] = [];

  showBlankModal = false;
  showInteractiveModal = false;

  selectedBlankDashboard?: Dashboardd;
  selectedInteractiveDashboard?: Dashboardd;

  selectedDashboard: Dashboardd | null = null;

  @Output() closeDashboard = new EventEmitter<void>();
  @Output() selectDashboard = new EventEmitter<string>();

  tableHeaders = [
    { key: 'name', label: 'Name' },
    { key: 'description', label: 'Description' },
    { key: 'createdBy', label: 'Created By' },
    { key: 'createdDate', label: 'Created Date' },
    { key: 'modifiedBy', label: 'Modified By' },
    { key: 'modifiedDate', label: 'Modified Date' },
    { key: 'public', label: 'Public' },
    { key: 'action', label: 'Action' }
  ];

  constructor(private dashboardService: DashboardService) {}

  ngOnInit(): void {
    this.loadDashboards();
  }

  loadDashboards(): void {
    this.dashboardService.getDashboards().subscribe(data => {
      this.tableData = data;
    });
  }

  onEditDashboard(dashboard: Dashboardd): void {
    this.selectedDashboard = dashboard;

    if (this.isInteractiveDashboard(dashboard)) {
      this.selectedInteractiveDashboard = dashboard;
      this.showInteractiveModal = true;
    } else {
      this.selectedBlankDashboard = dashboard;
      this.showBlankModal = true;
    }
  }

  onDeleteDashboard(id: number): void {
    if (confirm('Are you sure you want to delete this dashboard?')) {
      this.dashboardService.deleteDashboard(id).subscribe(() => {
        this.loadDashboards();
      });
    }
  }

  isInteractiveDashboard(dashboard: Dashboardd): boolean {
    return !!(dashboard.model || dashboard.groupBy || dashboard.aggregation);
  }
}

import { Component, EventEmitter, OnInit, Output } from '@angular/core';
import { DashboardService } from 'src/app/services/dashboard.service';
import { Dashboard } from 'src/app/models/dashboard.model';

@Component({
  selector: 'app-manage-dashboard',
  templateUrl: './manage-dashboard.component.html',
  styleUrls: ['./manage-dashboard.component.css']
})
export class ManageDashboardComponent implements OnInit {

  tableData: Dashboard[] = [];
  selectedDashboard: Dashboard | null = null;

  dashboards = [
    {
      id: 'blank',
      name: 'Blank Dashboard',
      icon: '',
      description: 'Start with a blank canvas'
    },
    {
      id: 'interactive',
      name: 'Interactive Dashboard',
      icon: '',
      description: 'Pre-built interactive components'
    }
  ];

  tableHeaders = [
    { key: 'name', label: 'Name' },
    { key: 'description', label: 'Description' },
    { key: 'createdBy', label: 'Created By' },
    { key: 'createdDate', label: 'Created Date' },
    { key: 'modifiedBy', label: 'Modified By' },
    { key: 'modifiedDate', label: 'Modified Date' },
    { key: 'public', label: 'Public' },
    { key: 'action', label: 'Action' }
  ];

  showBlankDashboard = false;
  showInteractiveDashboard = false;

  selectedBlankDashboard?: Dashboard;
  selectedInteractiveDashboard?: Dashboard;

  @Output() closeDashboard = new EventEmitter<void>();
  @Output() selectDashboard = new EventEmitter<string>();

  constructor(private dashboardService: DashboardService) {}

  ngOnInit(): void {
    this.loadDashboards();
  }

  loadDashboards(): void {
    this.dashboardService.getDashboards().subscribe({
      next: (data) => this.tableData = data,
      error: () => alert('Failed to load dashboards')
    });
  }

  onSelectDashboard(dashboardType: string): void {
    if (dashboardType === 'blank') {
      this.showBlankDashboard = true;
      this.showInteractiveDashboard = false;
    } else if (dashboardType === 'interactive') {
      this.showInteractiveDashboard = true;
      this.showBlankDashboard = false;
    }
    this.selectDashboard.emit(dashboardType);
  }

  onBackToManage(): void {
    this.showBlankDashboard = false;
    this.showInteractiveDashboard = false;
    this.selectedBlankDashboard = undefined;
    this.selectedInteractiveDashboard = undefined;
  }

  onNewDashboard(): void {
    this.selectedDashboard = null;
    this.showBlankDashboard = true;
    this.showInteractiveDashboard = false;
  }

  onEditDashboard(dashboard: Dashboard): void {
    this.selectedDashboard = dashboard;
    if (this.isInteractiveDashboard(dashboard)) {
      this.selectedInteractiveDashboard = dashboard;
      this.showInteractiveDashboard = true;
    } else {
      this.selectedBlankDashboard = dashboard;
      this.showBlankDashboard = true;
    }
  }

  onDeleteDashboard(id: number): void {
    if (confirm('Are you sure you want to delete this dashboard?')) {
      this.dashboardService.deleteDashboard(id).subscribe({
        next: () => {
          this.tableData = this.tableData.filter(d => d.id !== id);
          alert('Dashboard deleted successfully.');
        },
        error: () => alert('Delete failed')
      });
    }
  }

  isInteractiveDashboard(dashboard: Dashboard): boolean {
    return !!(dashboard.model || dashboard.groupBy || dashboard.aggregation);
  }

  onClose(): void {
    this.closeDashboard.emit();
  }
}




<div class="manage-dashboard-container">
  <!-- Main Manage Dashboard View -->
  <div *ngIf="!showBlankDashboard && !showInteractiveDashboard">
    <!-- Header -->
    <div class="header">
      <h2 class="title">Manage Dashboard</h2>
      <button class="close-btn" (click)="onClose()">x</button>
    </div>

    <!-- New Dashboard Button -->
    <div class="new-dashboard-section">
      <button class="new-dashboard-btn" (click)="onNewDashboard()">
        <span class="plus-icon">+</span>
        New Dashboard
      </button>
    </div>

    <!-- Dashboard Types -->
    <div class="dashboard-types">
      <div class="dashboard-card"
           *ngFor="let dashboard of dashboards"
           (click)="onSelectDashboard(dashboard.id)">
        <div class="dashboard-icon">{{ dashboard.icon }}</div>
        <div class="dashboard-name">{{ dashboard.name }}</div>
      </div>
    </div>

    <!-- All Dashboards Section -->
    <div class="all-dashboards-section">
      <h3 class="section-title">All Dashboards</h3>
      <div class="table-container">
        <table class="dashboards-table">
          <thead>
            <tr>
              <th *ngFor="let header of tableHeaders">{{ header.label }}</th>
            </tr>
          </thead>
          <tbody>
            <tr *ngIf="tableData.length === 0" class="no-data-row">
              <td [attr.colspan]="tableHeaders.length" class="no-data-cell">No Rows To Show</td>
            </tr>
            <tr *ngFor="let row of tableData">
              <td *ngFor="let header of tableHeaders">
                <ng-container [ngSwitch]="header.key">
                  <span *ngSwitchCase="'createdDate'">{{ row.createdDate | date: 'short' }}</span>
                  <span *ngSwitchCase="'modifiedDate'">{{ row.modifiedDate | date: 'short' }}</span>
                  <span *ngSwitchCase="'public'">{{ row.public ? 'Yes' : 'No' }}</span>
                  <span *ngSwitchCase="'action'">
                    <button mat-icon-button color="primary" (click)="onEditDashboard(row)">
                      <mat-icon>edit</mat-icon>
                    </button>
                    <button mat-icon-button color="warn" (click)="onDeleteDashboard(row.id)">
                      <mat-icon>delete</mat-icon>
                    </button>
                  </span>
                  <span *ngSwitchDefault>{{ row[header.key] }}</span>
                </ng-container>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>

  <!-- Blank Dashboard View -->
  <div *ngIf="showBlankDashboard">
    <app-blank-dashboard
      [editData]="selectedBlankDashboard"
      (dashboardClose)="onBackToManage()">
    </app-blank-dashboard>
  </div>

  <!-- Interactive Dashboard View -->
  <div *ngIf="showInteractiveDashboard" class="dashboard-view">
    <div class="dashboard-header">
      <button class="back-btn" (click)="onBackToManage()">Back to Manage</button>
      <button class="close-btn" (click)="onClose()">x</button>
    </div>
    <app-interactive-dashboard
      *ngIf="showInteractiveDashboard"
      [editData]="selectedInteractiveDashboard"
      (dashboardClose)="onBackToManage()">
    </app-interactive-dashboard>
  </div>
</div>





action 
import { Component } from '@angular/core';
import { ICellRendererAngularComp } from 'ag-grid-angular';
import { CommonModule } from '@angular/common';
import { MatIconModule } from '@angular/material/icon';
import { MatButtonModule } from '@angular/material/button';

@Component({
  selector: 'app-action-cell-renderer',
  standalone: true,
  imports: [CommonModule, MatButtonModule, MatIconModule],
  template: `
    <button mat-icon-button color="primary" (click)="onEditClicked()">
      <mat-icon>edit</mat-icon>
    </button>
    <button mat-icon-button color="warn" (click)="onDeleteClicked()">
      <mat-icon>delete</mat-icon>
    </button>
  `
})
export class ActionCellRendererComponent implements ICellRendererAngularComp {
  params: any;

  agInit(params: any): void {
    this.params = params;
  }

  refresh(params: any): boolean {
    return false;
  }

  onEditClicked(): void {
    if (this.params?.onEditClicked) {
      this.params.onEditClicked(this.params.data);
    }
  }

  onDeleteClicked(): void {
    if (this.params?.onDeleteClicked) {
      this.params.onDeleteClicked(this.params.data.id);
    }
  }
}

manage ts
import { Component, EventEmitter, Output, OnInit, Inject } from '@angular/core';
import { isPlatformBrowser, CommonModule } from '@angular/common';
import { PLATFORM_ID } from '@angular/core';
import { DashboardService } from './dashboard.service';
import { Dashboardd } from './dashboard.model';
import { ColDef } from 'ag-grid-community';
import { AgGridModule } from 'ag-grid-angular';
import { MatIconModule } from '@angular/material/icon';
import { MatButtonModule } from '@angular/material/button';
import { BlankDashboardComponent } from './blank-dashboard/blank-dashboard.component';
import { InteractiveDashboardComponent } from './interactive-dashboard/interactive-dashboard.component';
import { ActionCellRendererComponent } from './action-cell-renderer.component';
import { NgIf } from '@angular/common';

@Component({
  selector: 'app-manage-dashboard',
  standalone: true,
  templateUrl: './manage-dashboard.component.html',
  styleUrls: ['./manage-dashboard.component.css'],
  imports: [
    CommonModule,
    NgIf,
    MatIconModule,
    MatButtonModule,
    AgGridModule,
    BlankDashboardComponent,
    InteractiveDashboardComponent,
    ActionCellRendererComponent
  ]
})
export class ManageDashboardComponent implements OnInit {
  @Output() closeDashboard = new EventEmitter<void>();
  @Output() selectDashboard = new EventEmitter<string>();

  tableData: Dashboardd[] = [];
  selectedDashboard: Dashboardd | null = null;
  editingDashboard: Dashboardd | null = null;

  showBlankDashboard = false;
  showInteractiveDashboard = false;

  selectedBlankDashboard?: Dashboardd;
  selectedInteractiveDashboard?: Dashboardd;

  isBrowser: boolean;

  dashboards = [
    { id: 'blank', name: 'Blank Dashboard', icon: '' },
    { id: 'interactive', name: 'Interactive Dashboard', icon: '' }
  ];

  columnDefs: ColDef[] = [
    { field: 'name', headerName: 'Name' },
    { field: 'description', headerName: 'Description' },
    { field: 'createdBy', headerName: 'Created By' },
    { field: 'createdDate', headerName: 'Created Date', valueFormatter: this.dateFormatter },
    { field: 'modifiedBy', headerName: 'Modified By' },
    { field: 'modifiedDate', headerName: 'Modified Date', valueFormatter: this.dateFormatter },
    { field: 'public', headerName: 'Public', valueFormatter: this.booleanFormatter },
    {
      headerName: 'Action',
      field: 'action',
      cellRenderer: 'appActionCellRenderer',
      cellRendererParams: {
        onEditClicked: this.onEditDashboard.bind(this),
        onDeleteClicked: this.onDeleteDashboard.bind(this)
      },
      width: 120,
      pinned: 'right'
    }
  ];

  defaultColDef: ColDef = {
    flex: 1,
    resizable: true
  };

  frameworkComponents = {
    appActionCellRenderer: ActionCellRendererComponent
  };

  constructor(
    private dashboardService: DashboardService,
    @Inject(PLATFORM_ID) platformId: Object
  ) {
    this.isBrowser = isPlatformBrowser(platformId);
  }

  ngOnInit(): void {
    this.loadDashboards();
  }

  loadDashboards(): void {
    this.dashboardService.getDashboards().subscribe({
      next: (data) => {
        this.tableData = data.map(d => ({ ...d, action: '' }));
      },
      error: (error) => {
        console.error('Failed to load dashboards:', error);
        alert('Failed to load dashboards');
      }
    });
  }

  onSelectDashboard(dashboardType: string): void {
    if (dashboardType === 'blank') {
      this.showBlankDashboard = true;
      this.showInteractiveDashboard = false;
    } else if (dashboardType === 'interactive') {
      this.showInteractiveDashboard = true;
      this.showBlankDashboard = false;
    }
    this.selectDashboard.emit(dashboardType);
  }

  onNewDashboard(): void {
    this.selectedDashboard = null;
    this.showBlankDashboard = true;
    this.showInteractiveDashboard = false;
  }

  onEditDashboard(dashboard: Dashboardd): void {
    this.editingDashboard = dashboard;
    this.selectedDashboard = dashboard;

    if (this.isInteractiveDashboard(dashboard)) {
      this.selectedInteractiveDashboard = dashboard;
      this.showInteractiveDashboard = true;
      this.showBlankDashboard = false;
    } else {
      this.selectedBlankDashboard = dashboard;
      this.showBlankDashboard = true;
      this.showInteractiveDashboard = false;
    }
  }

  onDeleteDashboard(id: number): void {
    if (confirm('Are you sure you want to delete this dashboard?')) {
      this.dashboardService.deleteDashboard(id).subscribe({
        next: () => {
          this.tableData = this.tableData.filter(d => d.id !== id);
          alert('Dashboard deleted successfully.');
        },
        error: () => alert('Delete failed')
      });
    }
  }

  onBackToManage(): void {
    this.showBlankDashboard = false;
    this.showInteractiveDashboard = false;
    this.selectedBlankDashboard = undefined;
    this.selectedInteractiveDashboard = undefined;
    this.loadDashboards();
  }

  onClose(): void {
    this.closeDashboard.emit();
  }

  onDashboardCreated(): void {
    this.loadDashboards();
  }

  onGridReady(params: any): void {
    console.log('Grid is ready:', params);
  }

  dateFormatter(params: any): string {
    return new Date(params.value).toLocaleDateString();
  }

  booleanFormatter(params: any): string {
    return params.value ? 'Yes' : 'No';
  }

  isInteractiveDashboard(dashboard: Dashboardd): boolean {
    return !!(dashboard.model || dashboard.groupBy || dashboard.aggregation);
  }
}

manage html
<div class="manage-dashboard-container">
  <!-- Main Manage Dashboard View -->
  <div *ngIf="!showBlankDashboard && !showInteractiveDashboard">
    <!-- Header -->
    <div class="header">
      <h2 class="title">Manage Dashboard</h2>
      <button class="close-btn" (click)="onClose()">x</button>
    </div>

    <!-- New Dashboard Button -->
    <div class="new-dashboard-section">
      <button class="new-dashboard-btn" (click)="onNewDashboard()">
        <span class="plus-icon">+</span>
        New Dashboard
      </button>
    </div>

    <!-- Dashboard Types -->
    <div class="dashboard-types">
      <div class="dashboard-card" *ngFor="let dashboard of dashboards" (click)="onSelectDashboard(dashboard.id)">
        <div class="dashboard-icon">{{ dashboard.icon }}</div>
        <div class="dashboard-name">{{ dashboard.name }}</div>
      </div>
    </div>

    <!-- All Dashboards Section -->
    <div class="all-dashboards-section">
      <h3 class="section-title">All Dashboards</h3>
      <div class="table-container">
        <ng-container *ngIf="isBrowser; else placeholder">
          <ag-grid-angular
            class="ag-theme-balham"
            style="width: 100%; height: 500px;"
            [rowData]="tableData"
            [columnDefs]="columnDefs"
            [defaultColDef]="defaultColDef"
            [frameworkComponents]="frameworkComponents"
            (gridReady)="onGridReady($event)">
          </ag-grid-angular>
        </ng-container>
        <ng-template #placeholder>
          <app-ag-grid-placeholder></app-ag-grid-placeholder>
        </ng-template>
      </div>
    </div>
  </div>

  <!-- Blank Dashboard View -->
  <div *ngIf="showBlankDashboard">
    <app-blank-dashboard
      [editData]="selectedBlankDashboard"
      (dashboardClose)="onBackToManage()">
    </app-blank-dashboard>
  </div>

  <!-- Interactive Dashboard View -->
  <div *ngIf="showInteractiveDashboard">
    <div class="dashboard-header">
      <button class="back-btn" (click)="onBackToManage()">Back to Manage</button>
      <button class="close-btn" (click)="onClose()">x</button>
    </div>
    <app-interactive-dashboard
      [editData]="selectedInteractiveDashboard"
      (dashboardCreated)="onDashboardCreated()"
      (dashboardClose)="onBackToManage()">
    </app-interactive-dashboard>
  </div>
</div>




Here’s a full working version of your ManageDashboardComponent using Angular standalone components, including AgGridModule.withComponents() and ActionCellRendererComponent integration.


---

✅ manage-dashboard.component.ts

import { Component, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { AgGridModule } from 'ag-grid-angular';
import { ColDef, GridReadyEvent } from 'ag-grid-community';
import { MatIconModule } from '@angular/material/icon';
import { MatButtonModule } from '@angular/material/button';
import { CUSTOM_ELEMENTS_SCHEMA } from '@angular/core';

import { ActionCellRendererComponent } from './action-cell-renderer.component'; // Update path if needed

@Component({
  selector: 'app-manage-dashboard',
  standalone: true,
  templateUrl: './manage-dashboard.component.html',
  styleUrls: ['./manage-dashboard.component.css'],
  imports: [
    CommonModule,
    MatButtonModule,
    MatIconModule,
    AgGridModule.withComponents([ActionCellRendererComponent]),
    ActionCellRendererComponent,
  ],
  schemas: [CUSTOM_ELEMENTS_SCHEMA],
})
export class ManageDashboardComponent implements OnInit {
  tableData: any[] = []; // Populate with your dashboard data

  columnDefs: ColDef[] = [
    { headerName: 'Name', field: 'name' },
    { headerName: 'Type', field: 'type' },
    {
      headerName: 'Actions',
      field: 'actions',
      cellRenderer: 'appActionCellRenderer',
      cellRendererParams: {
        onEditClicked: this.onEditDashboard.bind(this),
        onDeleteClicked: this.onDeleteDashboard.bind(this),
      },
    },
  ];

  defaultColDef: ColDef = {
    flex: 1,
    sortable: true,
    filter: true,
    resizable: true,
  };

  frameworkComponents = {
    appActionCellRenderer: ActionCellRendererComponent,
  };

  isBrowser = true;

  ngOnInit(): void {
    // load data here if needed
  }

  onGridReady(event: GridReadyEvent): void {
    // optional grid setup
  }

  onEditDashboard(dashboard: any): void {
    console.log('Edit clicked', dashboard);
    // handle edit logic
  }

  onDeleteDashboard(dashboardId: string): void {
    console.log('Delete clicked', dashboardId);
    // handle delete logic
  }
}


---

✅ action-cell-renderer.component.ts

import { Component } from '@angular/core';
import { ICellRendererAngularComp } from 'ag-grid-angular';
import { CommonModule } from '@angular/common';
import { MatIconModule } from '@angular/material/icon';
import { MatButtonModule } from '@angular/material/button';

@Component({
  selector: 'app-action-cell-renderer',
  standalone: true,
  template: `
    <button mat-icon-button color="primary" (click)="onEditClicked()">
      <mat-icon>edit</mat-icon>
    </button>
    <button mat-icon-button color="warn" (click)="onDeleteClicked()">
      <mat-icon>delete</mat-icon>
    </button>
  `,
  imports: [CommonModule, MatButtonModule, MatIconModule],
})
export class ActionCellRendererComponent implements ICellRendererAngularComp {
  params: any;

  agInit(params: any): void {
    this.params = params;
  }

  refresh(): boolean {
    return false;
  }

  onEditClicked(): void {
    this.params.onEditClicked(this.params.data);
  }

  onDeleteClicked(): void {
    this.params.onDeleteClicked(this.params.data.id);
  }
}______________________

Thanks! Based on all your shared context, here is a clean and working setup for:


---

✅ manage-dashboard.component.html

<div class="all-dashboards-section">
  <h3 class="section-title">All Dashboards</h3>
  <div class="table-container">
    <ng-container *ngIf="isBrowser; else placeholder">
      <ag-grid-angular
        class="ag-theme-balham"
        style="width: 100%; height: 500px;"
        [rowData]="tableData"
        [columnDefs]="columnDefs"
        [defaultColDef]="defaultColDef"
        [frameworkComponents]="frameworkComponents"
        (gridReady)="onGridReady($event)">
      </ag-grid-angular>
    </ng-container>
    <ng-template #placeholder>
      <app-ag-grid-placeholder></app-ag-grid-placeholder>
    </ng-template>
  </div>
</div>


---

✅ manage-dashboard.component.ts

import { Component, inject } from '@angular/core';
import { CommonModule, isPlatformBrowser } from '@angular/common';
import { AgGridModule } from 'ag-grid-angular';
import { ColDef } from 'ag-grid-community';
import { DashboardService } from '../../services/dashboard.service';
import { ActionCellRendererComponent } from '../action-cell-renderer/action-cell-renderer.component';
import { AgGridPlaceholderComponent } from '../ag-grid-placeholder/ag-grid-placeholder.component';
import { PLATFORM_ID } from '@angular/core';

@Component({
  selector: 'app-manage-dashboard',
  standalone: true,
  imports: [CommonModule, AgGridModule, ActionCellRendererComponent, AgGridPlaceholderComponent],
  templateUrl: './manage-dashboard.component.html',
  styleUrls: ['./manage-dashboard.component.css']
})
export class ManageDashboardComponent {
  private dashboardService = inject(DashboardService);
  private platformId = inject(PLATFORM_ID);

  isBrowser = isPlatformBrowser(this.platformId);

  tableData: any[] = [];

  frameworkComponents = {
    actionCellRenderer: ActionCellRendererComponent
  };

  columnDefs: ColDef[] = [
    { field: 'name', headerName: 'Name' },
    { field: 'description', headerName: 'Description' },
    { field: 'createdBy', headerName: 'Created By' },
    { field: 'createdDate', headerName: 'Created Date', valueFormatter: this.dateFormatter },
    { field: 'isPublic', headerName: 'Public', valueFormatter: this.booleanFormatter },
    {
      field: 'actions',
      cellRenderer: 'actionCellRenderer',
      headerName: 'Actions',
      cellRendererParams: {
        onEditClicked: (data: any) => this.editDashboard(data),
        onDeleteClicked: (id: number) => this.deleteDashboard(id)
      }
    }
  ];

  defaultColDef: ColDef = {
    sortable: true,
    filter: true,
    resizable: true
  };

  ngOnInit() {
    this.loadDashboards();
  }

  onGridReady(params: any) {
    params.api.sizeColumnsToFit();
  }

  loadDashboards() {
    this.dashboardService.getDashboards().subscribe(data => {
      this.tableData = data;
    });
  }

  editDashboard(data: any) {
    // Logic to open edit modal and populate with data
    console.log('Edit dashboard', data);
  }

  deleteDashboard(id: number) {
    this.dashboardService.deleteDashboard(id).subscribe(() => {
      this.loadDashboards();
    });
  }

  dateFormatter(params: any): string {
    return new Date(params.value).toLocaleDateString();
  }

  booleanFormatter(params: any): string {
    return params.value ? 'Yes' : 'No';
  }
}


---

✅ action-cell-renderer.component.ts

import { Component } from '@angular/core';
import { ICellRendererAngularComp } from 'ag-grid-angular';
import { MatIconModule } from '@angular/material/icon';
import { MatButtonModule } from '@angular/material/button';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-action-cell-renderer',
  standalone: true,
  imports: [CommonModule, MatButtonModule, MatIconModule],
  template: `
    <button mat-icon-button color="primary" (click)="onEditClicked()">
      <mat-icon>edit</mat-icon>
    </button>
    <button mat-icon-button color="warn" (click)="onDeleteClicked()">
      <mat-icon>delete</mat-icon>
    </button>
  `
})
export class ActionCellRendererComponent implements ICellRendererAngularComp {
  params: any;

  agInit(params: any): void {
    this.params = params;
  }

  refresh(): boolean {
    return false;
  }

  onEditClicked(): void {
    this.params.onEditClicked(this.params.data);
  }

  onDeleteClicked(): void {
    this.params.onDeleteClicked(this.params.data.id);
  }
}


---

✅ ag-grid-placeholder.component.ts (Optional fallback when not browser)

import { Component } from '@angular/core';

@Component({
  selector: 'app-ag-grid-placeholder',
  standalone: true,
  template: `<p>Loading dashboard table...</p>`
})
export class AgGridPlaceholderComponent {}


---

Would you like the working dashboard.model.ts and dashboard.service.ts cleaned up too?


import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Dashboardd } from './dashboard.model';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class DashboardService {
  private apiUrl = 'http://localhost:8083/api/dashboards';

  constructor(private http: HttpClient) {}

  getDashboards(): Observable<Dashboardd[]> {
    console.log('Calling API:', this.apiUrl);
    return this.http.get<Dashboardd[]>(this.apiUrl);
  }

  addDashboard(dashboard: Dashboardd): Observable<Dashboardd> {
    return this.http.post<Dashboardd>(this.apiUrl, dashboard);
  }

  deleteDashboard(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }

  updateDashboard(id: number, dashboard: Dashboardd): Observable<Dashboardd> {
    return this.http.put<Dashboardd>(`${this.apiUrl}/${id}`, dashboard);
  }
}



manage Comp html
import { Component, OnInit, inject } from '@angular/core';
import { ColDef, GridReadyEvent } from 'ag-grid-community';
import { DashboardService } from '../../../services/dashboard.service';
import { Dashboardd } from '../../../services/dashboard.model';
import { ActionCellRendererComponent } from './action-cell-renderer.component';
import { AgGridAngular } from 'ag-grid-angular';

@Component({
  selector: 'app-manage-dashboard',
  standalone: true,
  imports: [AgGridAngular, ActionCellRendererComponent],
  templateUrl: './manage-dashboard.component.html'
})
export class ManageDashboardComponent implements OnInit {
  dashboardService = inject(DashboardService);

  tableData: Dashboardd[] = [];
  columnDefs: ColDef[] = [];
  defaultColDef: ColDef = {
    flex: 1,
    sortable: true,
    filter: true,
    resizable: true
  };
  frameworkComponents: any;
  selectedDashboard: Dashboardd | null = null;

  showBlankModal = false;
  showInteractiveModal = false;
  isEditingBlank = false;
  isEditingInteractive = false;

  isBrowser = typeof window !== 'undefined';

  ngOnInit(): void {
    this.frameworkComponents = {
      actionCellRenderer: ActionCellRendererComponent
    };

    this.columnDefs = [
      { field: 'id' },
      { field: 'name' },
      { field: 'description' },
      { field: 'createdBy' },
      { field: 'modifiedBy' },
      {
        headerName: 'Actions',
        cellRenderer: 'actionCellRenderer',
        cellRendererParams: {
          onEditClicked: this.onEditClicked.bind(this),
          onDeleteClicked: this.onDeleteClicked.bind(this)
        }
      }
    ];

    this.loadDashboards();
  }

  loadDashboards(): void {
    this.dashboardService.getDashboards().subscribe(data => {
      this.tableData = data;
    });
  }

  onGridReady(params: GridReadyEvent): void {
    params.api.sizeColumnsToFit();
  }

  onEditClicked(dashboard: Dashboardd): void {
    this.selectedDashboard = dashboard;

    const isInteractive = !!(
      dashboard.model || dashboard.groupBy || dashboard.aggregation || dashboard.aggregationField
    );

    if (isInteractive) {
      this.isEditingInteractive = true;
      this.isEditingBlank = false;
      this.showInteractiveModal = true;
    } else {
      this.isEditingBlank = true;
      this.isEditingInteractive = false;
      this.showBlankModal = true;
    }
  }

  onDeleteClicked(id: number): void {
    this.dashboardService.deleteDashboard(id).subscribe(() => {
      this.loadDashboards();
    });
  }

  closeBlankModal(): void {
    this.showBlankModal = false;
    this.selectedDashboard = null;
    this.loadDashboards();
  }

  closeInteractiveModal(): void {
    this.showInteractiveModal = false;
    this.selectedDashboard = null;
    this.loadDashboards();
  }
}

ts
import { Component, OnInit } from '@angular/core';
import { ColDef } from 'ag-grid-community';
import { DashboardService } from '../dashboard.service';
import { Dashboardd } from '../dashboard.model';
import { ActionCellRendererComponent } from '../action-cell-renderer.component';

@Component({
  selector: 'app-manage-dashboard',
  templateUrl: './manage-dashboard.component.html',
  standalone: true,
  imports: [ActionCellRendererComponent],
})
export class ManageDashboardComponent implements OnInit {
  tableData: Dashboardd[] = [];
  columnDefs: ColDef[] = [
    { field: 'name', headerName: 'Name' },
    { field: 'description', headerName: 'Description' },
    { field: 'createdBy', headerName: 'Created By' },
    { field: 'createdDate', headerName: 'Created Date', valueFormatter: this.dateFormatter },
    { field: 'modifiedBy', headerName: 'Modified By' },
    { field: 'modifiedDate', headerName: 'Modified Date', valueFormatter: this.dateFormatter },
    { field: 'isPublic', headerName: 'Public', valueFormatter: this.booleanFormatter },
    {
      headerName: 'Actions',
      cellRenderer: ActionCellRendererComponent,
      cellRendererParams: {
        onEditClicked: (dashboard: Dashboardd) => this.onEditClicked(dashboard),
        onDeleteClicked: (id: number) => this.onDeleteClicked(id),
      },
    }
  ];

  defaultColDef: ColDef = {
    resizable: true,
    sortable: true,
    filter: true,
  };

  frameworkComponents = {
    actionCellRenderer: ActionCellRendererComponent,
  };

  isBrowser = typeof window !== 'undefined';
  showBlankModal = false;
  showInteractiveModal = false;
  isEditingBlank = false;
  isEditingInteractive = false;
  selectedDashboard: Dashboardd | null = null;

  constructor(private dashboardService: DashboardService) {}

  ngOnInit(): void {
    this.loadDashboards();
  }

  loadDashboards(): void {
    this.dashboardService.getDashboards().subscribe(data => {
      this.tableData = data;
    });
  }

  onGridReady(): void {}

  dateFormatter(params: any): string {
    return new Date(params.value).toLocaleDateString();
  }

  booleanFormatter(params: any): string {
    return params.value ? 'Yes' : 'No';
  }

  onEditClicked(dashboard: Dashboardd): void {
    this.selectedDashboard = dashboard;
    const isInteractive = !!(dashboard.model || dashboard.groupBy || dashboard.aggregation || dashboard.aggregationField);

    if (isInteractive) {
      this.isEditingInteractive = true;
      this.isEditingBlank = false;
      this.showInteractiveModal = true;
    } else {
      this.isEditingBlank = true;
      this.isEditingInteractive = false;
      this.showBlankModal = true;
    }
  }

  onDeleteClicked(id: number): void {
    this.dashboardService.deleteDashboard(id).subscribe(() => {
      this.loadDashboards();
    });
  }

  onModalClosed(): void {
    this.showBlankModal = false;
    this.showInteractiveModal = false;
    this.selectedDashboard = null;
    this.isEditingBlank = false;
    this.isEditingInteractive = false;
    this.loadDashboards();
  }
}

html
<div class="all-dashboards-section">
  <h3 class="section-title">All Dashboards</h3>

  <div class="table-container">
    <ng-container *ngIf="isBrowser; else placeholder">
      <ag-grid-angular
        class="ag-theme-balham"
        style="width: 100%; height: 500px;"
        [rowData]="tableData"
        [columnDefs]="columnDefs"
        [defaultColDef]="defaultColDef"
        [frameworkComponents]="frameworkComponents"
        (gridReady)="onGridReady($event)">
      </ag-grid-angular>
    </ng-container>

    <ng-template #placeholder>
      <app-ag-grid-placeholder></app-ag-grid-placeholder>
    </ng-template>
  </div>

  <ng-container *ngIf="showBlankModal">
    <app-blank-dashboard
      [dashboardData]="selectedDashboard"
      [isEditMode]="isEditingBlank"
      (closeModal)="onModalClosed()">
    </app-blank-dashboard>
  </ng-container>

  <ng-container *ngIf="showInteractiveModal">
    <app-interactive-dashboard
      [dashboardData]="selectedDashboard"
      [isEditMode]="isEditingInteractive"
      (closeModal)="onModalClosed()">
    </app-interactive-dashboard>
  </ng-container>
</div>

dashboard service ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Dashboardd } from './dashboard.model';
import { Observable } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class DashboardService {
  private apiUrl = 'http://localhost:8083/api/dashboards';

  constructor(private http: HttpClient) {}

  getDashboards(): Observable<Dashboardd[]> {
    return this.http.get<Dashboardd[]>(this.apiUrl);
  }

  addDashboard(dashboard: Dashboardd): Observable<Dashboardd> {
    return this.http.post<Dashboardd>(this.apiUrl, dashboard);
  }

  updateDashboard(id: number, dashboard: Dashboardd): Observable<Dashboardd> {
    return this.http.put<Dashboardd>(`${this.apiUrl}/${id}`, dashboard);
  }

  deleteDashboard(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}

action cell ts
import { Component } from '@angular/core';
import { ICellRendererAngularComp } from 'ag-grid-angular';
import { MatIconModule } from '@angular/material/icon';
import { MatButtonModule } from '@angular/material/button';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-action-cell-renderer',
  standalone: true,
  imports: [CommonModule, MatButtonModule, MatIconModule],
  template: `
    <button mat-icon-button color="primary" (click)="onEditClicked()">
      <mat-icon>edit</mat-icon>
    </button>
    <button mat-icon-button color="warn" (click)="onDeleteClicked()">
      <mat-icon>delete</mat-icon>
    </button>
  `
})
export class ActionCellRendererComponent implements ICellRendererAngularComp {
  params: any;

  agInit(params: any): void {
    this.params = params;
  }

  refresh(): boolean {
    return false;
  }

  onEditClicked(): void {
    this.params.onEditClicked(this.params.data);
  }

  onDeleteClicked(): void {
    this.params.onDeleteClicked(this.params.data.id);
  }
}

ag grid ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-ag-grid-placeholder',
  standalone: true,
  template: `<div class="placeholder">Loading Grid...</div>`,
})
export class AgGridPlaceholderComponent {}






import { Component, OnInit } from '@angular/core';
import { DashboardService } from '../dashboard.service';
import { Dashboardd } from '../dashboard.model';
import { ColDef } from 'ag-grid-community';
import { ActionCellRendererComponent } from '../action-cell-renderer.component';

@Component({
  selector: 'app-manage-dashboard',
  templateUrl: './manage-dashboard.component.html',
  standalone: true,
  imports: [ActionCellRendererComponent],
})
export class ManageDashboardComponent implements OnInit {
  tableData: Dashboardd[] = [];

  showBlankDashboard = false;
  showInteractiveDashboard = false;

  selectedBlankDashboard: Dashboardd | null = null;
  selectedInteractiveDashboard: Dashboardd | null = null;

  columnDefs: ColDef[] = [
    { field: 'name', headerName: 'Name' },
    { field: 'description', headerName: 'Description' },
    { field: 'createdBy', headerName: 'Created By' },
    { field: 'createdDate', headerName: 'Created Date', valueFormatter: this.dateFormatter },
    { field: 'modifiedBy', headerName: 'Modified By' },
    { field: 'modifiedDate', headerName: 'Modified Date', valueFormatter: this.dateFormatter },
    { field: 'isPublic', headerName: 'Public', valueFormatter: this.booleanFormatter },
    {
      headerName: 'Actions',
      cellRenderer: ActionCellRendererComponent,
      cellRendererParams: {
        onEditClicked: (dashboard: Dashboardd) => this.onEditClicked(dashboard),
        onDeleteClicked: (id: number) => this.onDeleteClicked(id),
      },
    }
  ];

  defaultColDef: ColDef = {
    resizable: true,
    sortable: true,
    filter: true,
  };

  constructor(private dashboardService: DashboardService) {}

  ngOnInit(): void {
    this.loadDashboards();
  }

  loadDashboards(): void {
    this.dashboardService.getDashboards().subscribe(data => {
      this.tableData = data;
    });
  }

  dateFormatter(params: any): string {
    return new Date(params.value).toLocaleDateString();
  }

  booleanFormatter(params: any): string {
    return params.value ? 'Yes' : 'No';
  }

  onEditClicked(dashboard: Dashboardd): void {
    const isInteractive = !!(dashboard.model || dashboard.groupBy || dashboard.aggregation || dashboard.aggregationField);

    if (isInteractive) {
      this.selectedInteractiveDashboard = dashboard;
      this.showInteractiveDashboard = true;
    } else {
      this.selectedBlankDashboard = dashboard;
      this.showBlankDashboard = true;
    }
  }

  onDeleteClicked(id: number): void {
    this.dashboardService.deleteDashboard(id).subscribe(() => {
      this.loadDashboards();
    });
  }

  onDashboardCreated(): void {
    this.loadDashboards();
    this.onBackToManage();
  }

  onBackToManage(): void {
    this.showBlankDashboard = false;
    this.showInteractiveDashboard = false;
    this.selectedBlankDashboard = null;
    this.selectedInteractiveDashboard = null;
  }

  onClose(): void {
    this.onBackToManage();
  }
}



<div class="all-dashboards-section">
  <h3 class="section-title">All Dashboards</h3>

  <!-- Main Dashboard Table -->
  <div class="table-container" *ngIf="!showBlankDashboard && !showInteractiveDashboard">
    <ag-grid-angular
      class="ag-theme-balham"
      style="width: 100%; height: 500px;"
      [rowData]="tableData"
      [columnDefs]="columnDefs"
      [defaultColDef]="defaultColDef">
    </ag-grid-angular>
  </div>

  <!-- Blank Dashboard View -->
  <div *ngIf="showBlankDashboard">
    <app-blank-dashboard
      [editData]="selectedBlankDashboard"
      (dashboardClose)="onBackToManage()">
    </app-blank-dashboard>
  </div>

  <!-- Interactive Dashboard View -->
  <div *ngIf="showInteractiveDashboard">
    <div class="dashboard-header">
      <button class="back-btn" (click)="onBackToManage()">Back to Manage</button>
      <button class="close-btn" (click)="onClose()">x</button>
    </div>
    <app-interactive-dashboard
      [editData]="selectedInteractiveDashboard"
      (dashboardCreated)="onDashboardCreated()"
      (dashboardClose)="onBackToManage()">
    </app-interactive-dashboard>
  </div>
</div>

__________________
To make your current ManageDashboardComponent look like Image 1 while preserving the ag-grid and incorporating your provided HTML/CSS (from Image 2), here is a cleaned-up and corrected version of the component HTML and associated CSS structure.


---

✅ Final manage-dashboard.component.html (converted like Image 1):

<div class="manage-dashboard-container">
  <!-- Header -->
  <div class="header">
    <h1 class="title">Manage Dashboard</h1>
    <button class="close-btn" (click)="onClose()">
      <mat-icon>close</mat-icon>
    </button>
  </div>

  <!-- New Dashboard Buttons -->
  <div class="new-dashboard-section">
    <div class="dashboard-types">
      <div class="dashboard-card" (click)="onNewBlankDashboard()">
        <mat-icon class="dashboard-icon">speed</mat-icon>
        <div class="dashboard-name">Blank Dashboard</div>
      </div>
      <div class="dashboard-card" (click)="onNewInteractiveDashboard()">
        <mat-icon class="dashboard-icon">insights</mat-icon>
        <div class="dashboard-name">Interactive Dashboard</div>
      </div>
    </div>
  </div>

  <!-- All Dashboards Table -->
  <div class="all-dashboards-section">
    <h2 class="section-title">All Dashboards</h2>
    <div class="table-container">
      <ag-grid-angular
        class="ag-theme-alpine"
        style="width: 100%; height: 500px;"
        [gridOptions]="gridOptions"
        [rowData]="tableData"
        (gridReady)="onGridReady($event)">
      </ag-grid-angular>
    </div>
  </div>

  <!-- Modals -->
  <app-blank-dashboard
    *ngIf="showBlankDashboard"
    [editData]="selectedBlankDashboard"
    (dashboardClose)="onClose()"
    (dashboardCreated)="onDashboardCreated()">
  </app-blank-dashboard>

  <app-interactive-dashboard
    *ngIf="showInteractiveDashboard"
    [editData]="selectedInteractiveDashboard"
    (dashboardClose)="onClose()"
    (dashboardCreated)="onDashboardCreated()">
  </app-interactive-dashboard>
</div>


---

✅ Updated manage-dashboard.component.scss (Styled like Image 1):

.manage-dashboard-container {
  padding: 20px;
  background-color: #f5f5f5;
  min-height: 100vh;
  font-family: Arial, sans-serif;

  .header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
    background-color: white;
    padding: 15px 20px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);

    .title {
      margin: 0;
      font-size: 20px;
      font-weight: 600;
      color: #333;
    }

    .close-btn {
      background: none;
      border: none;
      font-size: 24px;
      cursor: pointer;
      color: #666;
      padding: 0;
      width: 30px;
      height: 30px;
      display: flex;
      align-items: center;
      justify-content: center;
      border-radius: 50%;
      transition: background-color 0.2s;

      &:hover {
        background-color: #f0f0f0;
        color: #333;
      }
    }
  }

  .new-dashboard-section {
    margin-bottom: 30px;

    .dashboard-types {
      display: flex;
      gap: 20px;

      .dashboard-card {
        background-color: white;
        border: 1px solid #e0e0e0;
        border-radius: 8px;
        padding: 30px 20px;
        text-align: center;
        cursor: pointer;
        transition: all 0.2s;
        min-width: 150px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
        display: flex;
        flex-direction: column;
        align-items: center;

        &:hover {
          border-color: #007bff;
          box-shadow: 0 4px 8px rgba(0, 123, 255, 0.2);
          transform: translateY(-2px);
        }

        .dashboard-icon {
          font-size: 60px;
          margin-bottom: 15px;
          color: #5ca0fa;
        }

        .dashboard-name {
          font-size: 14px;
          font-weight: 500;
          color: #333;
        }
      }
    }
  }

  .all-dashboards-section {
    background-color: white;
    border-radius: 8px;
    padding: 20px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);

    .section-title {
      margin: 0 0 20px 0;
      font-size: 16px;
      font-weight: 600;
      color: #333;
    }

    .table-container {
      overflow-x: auto;
      width: 100%;
      display: flex;
      justify-content: center;
    }
  }

  @media (max-width: 768px) {
    padding: 15px;

    .dashboard-types {
      flex-direction: column;
      gap: 15px;

      .dashboard-card {
        min-width: auto;
      }
    }

    .header {
      padding: 12px 15px;

      .title {
        font-size: 18px;
      }
    }
  }
}


---

✅ Notes for Integration:

Ensure your component has:

showBlankDashboard: boolean;
showInteractiveDashboard: boolean;
selectedBlankDashboard: Dashboardd | null;
selectedInteractiveDashboard: Dashboardd | null;

In the component .ts, define:

onNewBlankDashboard() {
  this.selectedBlankDashboard = null;
  this.showBlankDashboard = true;
  this.showInteractiveDashboard = false;
}

onNewInteractiveDashboard() {
  this.selectedInteractiveDashboard = null;
  this.showInteractiveDashboard = true;
  this.showBlankDashboard = false;
}


Would you like the updated .ts file included with complete methods for onClose, onDashboardCreated, onGridReady, etc. as well?



onDeleteClicked(dashboard: Dashboardd): void {
  if (confirm('Are you sure you want to delete this dashboard?')) {
    this.dashboardService.deleteDashboard(dashboard.id).subscribe({
      next: () => {
        console.log('Dashboard deleted');
        this.loadDashboards(); // 🔄 Refresh list
      },
      error: (err) => {
        console.error('Error deleting dashboard', err);
      },
    });
  }
}

loadDashboards(): void {
  this.dashboardService.getAllDashboards().subscribe({
    next: (data) => {
      this.blankDashboards = data.filter(d => d.type === 'blank');
      this.interactiveDashboards = data.filter(d => d.type === 'interactive');
    },
    error: (err) => {
      console.error('Error loading dashboards', err);
    },
  });
}
--------------------------
import {
  Component,
  EventEmitter,
  OnInit,
  Output,
  Renderer2,
  ElementRef
} from '@angular/core';
import { DashboardService } from './dashboard.service';
import { CommonModule } from '@angular/common';
import { MatIconModule } from '@angular/material/icon';
import { MatButtonModule } from '@angular/material/button';
import { InteractiveDashboardComponent } from './interactive-dashboard/interactive-dashboard.component';
import { BlankDashboardComponent } from './blank-dashboard/blank-dashboard.component';
import { Dashboardd } from './dashboard.model';
import { AgGridModule } from 'ag-grid-angular';
import { ColDef } from 'ag-grid-community';

@Component({
  selector: 'app-manage-dashboard',
  templateUrl: './manage-dashboard.component.html',
  styleUrls: ['./manage-dashboard.component.css'],
  standalone: true,
  imports: [
    CommonModule,
    MatIconModule,
    MatButtonModule,
    BlankDashboardComponent,
    InteractiveDashboardComponent,
    AgGridModule
  ]
})
export class ManageDashboardComponent implements OnInit {
  tableData: Dashboardd[] = [];
  showBlankDashboard = false;
  showInteractiveDashboard = false;
  selectedBlankDashboard?: Dashboardd;
  selectedInteractiveDashboard?: Dashboardd;

  @Output() closeDashboard = new EventEmitter<void>();
  @Output() selectDashboard = new EventEmitter<string>();

  columnDefs: ColDef[] = [
    { field: 'name', headerName: 'Name' },
    { field: 'description', headerName: 'Description' },
    { field: 'createdBy', headerName: 'Created By' },
    {
      field: 'createdDate',
      headerName: 'Created Date',
      valueFormatter: this.dateFormatter
    },
    { field: 'modifiedBy', headerName: 'Modified By' },
    {
      field: 'modifiedDate',
      headerName: 'Modified Date',
      valueFormatter: this.dateFormatter
    },
    {
      field: 'isPublic',
      headerName: 'Public',
      valueFormatter: this.booleanFormatter
    },
    {
      headerName: 'Actions',
      cellRenderer: this.actionCellRenderer.bind(this)
    }
  ];

  defaultColDef: ColDef = {
    resizable: true,
    sortable: true,
    filter: true
  };

  gridOptions: any = {
    columnDefs: this.columnDefs,
    defaultColDef: this.defaultColDef,
    rowData: this.tableData,
    onGridReady: this.onGridReady.bind(this),
    rowSelection: 'single',
    pagination: true,
    paginationPageSize: 10,
    suppressRowClickSelection: true,
    animateRows: true
  };

  constructor(
    private dashboardService: DashboardService,
    private renderer: Renderer2,
    private elementRef: ElementRef
  ) {}

  ngOnInit(): void {
    this.loadDashboards();
  }

  loadDashboards(): void {
    this.dashboardService.getDashboards().subscribe(
      (data: Dashboardd[]) => {
        this.tableData = data;
      },
      error => {
        console.error('Failed to load dashboards', error);
      }
    );
  }

  dateFormatter(params: any): string {
    return new Date(params.value).toLocaleDateString();
  }

  booleanFormatter(params: any): string {
    return params.value ? 'Yes' : 'No';
  }

  onEditClicked(dashboard: Dashboardd): void {
    const isInteractive = !!(
      dashboard.model &&
      dashboard.groupBy &&
      dashboard.aggregation &&
      dashboard.aggregationField &&
      dashboard.columnField
    );

    if (isInteractive) {
      this.selectedInteractiveDashboard = dashboard;
      this.showInteractiveDashboard = true;
      this.showBlankDashboard = false;
    } else {
      this.selectedBlankDashboard = dashboard;
      this.showBlankDashboard = true;
      this.showInteractiveDashboard = false;
    }
  }

  onDeleteClicked(id: number): void {
    this.dashboardService.deleteDashboard(id).subscribe(() => {
      this.loadDashboards();
    });
  }

  onDashboardCreated(): void {
    this.loadDashboards();
    this.onBackToManage();
  }

  onBackToManage(): void {
    this.showBlankDashboard = false;
    this.showInteractiveDashboard = false;
    this.selectedBlankDashboard = undefined;
    this.selectedInteractiveDashboard = undefined;
  }

  onClose(): void {
    this.onBackToManage();
  }

  toggleBlankDashboard(): void {
    this.showBlankDashboard = true;
    this.showInteractiveDashboard = false;
    this.selectedBlankDashboard = undefined;
  }

  toggleInteractiveDashboard(): void {
    this.showInteractiveDashboard = true;
    this.showBlankDashboard = false;
    this.selectedInteractiveDashboard = undefined;
  }

  onGridReady(params: any): void {
    params.api.sizeColumnsToFit();
  }

  actionCellRenderer(params: any): HTMLElement {
    const div = this.renderer.createElement('div');
    const editButton = this.renderer.createElement('button');
    const deleteButton = this.renderer.createElement('button');

    this.renderer.addClass(editButton, 'mat-icon-button');
    this.renderer.addClass(deleteButton, 'mat-icon-button');

    const editIcon = this.renderer.createElement('mat-icon');
    const deleteIcon = this.renderer.createElement('mat-icon');

    this.renderer.appendChild(editIcon, this.renderer.createText('edit'));
    this.renderer.appendChild(deleteIcon, this.renderer.createText('delete'));

    this.renderer.appendChild(editButton, editIcon);
    this.renderer.appendChild(deleteButton, deleteIcon);

    this.renderer.appendChild(div, editButton);
    this.renderer.appendChild(div, deleteButton);

    this.renderer.listen(editButton, 'click', () => {
      this.onEditClicked(params.data);
    });

    this.renderer.listen(deleteButton, 'click', () => {
      this.onDeleteClicked(params.data.id);
    });

    return div;
  }
}

html
<div class="manage-dashboard-container" style="width: 100vw;">
  <!-- Header -->
  <div class="header">
    <h1 class="title">Manage Dashboard</h1>
    <button class="close-btn" (click)="onClose()">
      <mat-icon>close</mat-icon>
    </button>
  </div>

  <!-- New Dashboard Buttons -->
  <div class="new-dashboard-section">
    <div class="dashboard-types">
      <div class="dashboard-card" (click)="toggleBlankDashboard()">
        <mat-icon class="dashboard-icon">speed</mat-icon>
        <div class="dashboard-name">Blank Dashboard</div>
      </div>

      <div class="dashboard-card" (click)="toggleInteractiveDashboard()">
        <mat-icon class="dashboard-icon">analytics</mat-icon>
        <div class="dashboard-name">Interactive Dashboard</div>
      </div>
    </div>
  </div>

  <!-- All Dashboards Table -->
  <div class="all-dashboards-section">
    <h2 class="section-title">All Dashboards</h2>
    <div class="table-container">
      <ag-grid-angular
        class="ag-theme-balham"
        style="width: 100%; height: 500px;"
        [gridOptions]="gridOptions"
        [rowData]="tableData"
        (gridReady)="onGridReady($event)"
      ></ag-grid-angular>
    </div>
  </div>

  <!-- Modals -->
  <app-blank-dashboard
    *ngIf="showBlankDashboard"
    [editData]="selectedBlankDashboard"
    (dashboardClose)="onClose()"
    (dashboardCreated)="onDashboardCreated()"
  ></app-blank-dashboard>

  <app-interactive-dashboard
    *ngIf="showInteractiveDashboard"
    [editData]="selectedInteractiveDashboard"
    (dashboardClose)="onClose()"
    (dashboardCreated)="onDashboardCreated()"
  ></app-interactive-dashboard>
</div>




// manage-dashboard.component.ts
import { Component, OnInit } from '@angular/core';
import { DashboardService } from '../dashboard.service';
import { Dashboardd } from '../dashboard.model';

@Component({
  selector: 'app-manage-dashboard',
  templateUrl: './manage-dashboard.component.html',
  styleUrls: ['./manage-dashboard.component.css'],
})
export class ManageDashboardComponent implements OnInit {
  tableData: Dashboardd[] = [];
  showBlankDashboard = false;
  showInteractiveDashboard = false;
  selectedBlankDashboard?: Dashboardd;
  selectedInteractiveDashboard?: Dashboardd;

  constructor(private dashboardService: DashboardService) {}

  ngOnInit(): void {
    this.loadDashboards();
  }

  loadDashboards(): void {
    this.dashboardService.getAllDashboards().subscribe((data) => {
      this.tableData = data;
    });
  }

  toggleBlankDashboard(): void {
    this.showBlankDashboard = true;
    this.showInteractiveDashboard = false;
    this.selectedBlankDashboard = undefined;
  }

  toggleInteractiveDashboard(): void {
    this.showInteractiveDashboard = true;
    this.showBlankDashboard = false;
    this.selectedInteractiveDashboard = undefined;
  }

  onEditClicked(dashboard: Dashboardd): void {
    const isInteractive = !!(
      dashboard.model &&
      dashboard.groupBy &&
      dashboard.aggregation &&
      dashboard.aggregationField &&
      dashboard.columnField
    );

    if (isInteractive) {
      this.selectedInteractiveDashboard = { ...dashboard };
      this.showInteractiveDashboard = true;
      this.showBlankDashboard = false;
    } else {
      this.selectedBlankDashboard = { ...dashboard };
      this.showBlankDashboard = true;
      this.showInteractiveDashboard = false;
    }
  }

  onDeleteClicked(id: number): void {
    if (confirm('Are you sure you want to delete this dashboard?')) {
      this.dashboardService.deleteDashboard(id).subscribe(() => {
        this.loadDashboards();
      });
    }
  }

  onDashboardCreated(): void {
    this.loadDashboards();
    this.onBackToManage();
  }

  onBackToManage(): void {
    this.showBlankDashboard = false;
    this.showInteractiveDashboard = false;
  }
}

<!-- manage-dashboard.component.html -->
<div class="all-dashboards-section">
  <h2 class="section-title">All Dashboards</h2>
  <div class="action-buttons">
    <button mat-button color="primary" (click)="toggleBlankDashboard()">+ New Blank Dashboard</button>
    <button mat-button color="accent" (click)="toggleInteractiveDashboard()">+ New Interactive Dashboard</button>
  </div>

  <div class="table-container" *ngIf="!showBlankDashboard && !showInteractiveDashboard">
    <ag-grid-angular
      class="ag-theme-balham"
      style="width: 100%; height: 500px;"
      [rowData]="tableData"
      [columnDefs]="[
        { headerName: 'Name', field: 'name' },
        { headerName: 'Type', field: 'type' },
        {
          headerName: 'Actions',
          cellRenderer: params => {
            return `<button (click)='onEditClicked(${params.data.id})'>Edit</button>
                    <button (click)='onDeleteClicked(${params.data.id})'>Delete</button>`;
          }
        }
      ]">
    </ag-grid-angular>
  </div>

  <app-blank-dashboard
    *ngIf="showBlankDashboard"
    [editData]="selectedBlankDashboard"
    (dashboardClose)="onBackToManage()">
  </app-blank-dashboard>

  <app-interactive-dashboard
    *ngIf="showInteractiveDashboard"
    [editData]="selectedInteractiveDashboard"
    (dashboardCreated)="onDashboardCreated()"
    (dashboardClose)="onBackToManage()">
  </app-interactive-dashboard>
</div>



import {
  Component,
  EventEmitter,
  OnInit,
  Output,
  Renderer2,
  ElementRef
} from '@angular/core';
import { DashboardService } from './dashboard.service';
import { CommonModule } from '@angular/common';
import { MatIconModule } from '@angular/material/icon';
import { MatButtonModule } from '@angular/material/button';
import { InteractiveDashboardComponent } from './interactive-dashboard/interactive-dashboard.component';
import { BlankDashboardComponent } from './blank-dashboard/blank-dashboard.component';
import { Dashboardd } from './dashboard.model';
import { AgGridModule } from 'ag-grid-angular';
import { ColDef } from 'ag-grid-community';

@Component({
  selector: 'app-manage-dashboard',
  templateUrl: './manage-dashboard.component.html',
  styleUrls: ['./manage-dashboard.component.css'],
  standalone: true,
  imports: [
    CommonModule,
    MatIconModule,
    MatButtonModule,
    BlankDashboardComponent,
    InteractiveDashboardComponent,
    AgGridModule
  ]
})
export class ManageDashboardComponent implements OnInit {
  tableData: Dashboardd[] = [];
  showBlankDashboard = false;
  showInteractiveDashboard = false;
  selectedBlankDashboard?: Dashboardd;
  selectedInteractiveDashboard?: Dashboardd;

  @Output() closeDashboard = new EventEmitter<void>();
  @Output() selectDashboard = new EventEmitter<string>();

  columnDefs: ColDef[] = [
    { field: 'name', headerName: 'Name' },
    { field: 'description', headerName: 'Description' },
    { field: 'createdBy', headerName: 'Created By' },
    { field: 'createdDate', headerName: 'Created Date', valueFormatter: this.dateFormatter },
    { field: 'modifiedBy', headerName: 'Modified By' },
    { field: 'modifiedDate', headerName: 'Modified Date', valueFormatter: this.dateFormatter },
    { field: 'isPublic', headerName: 'Public', valueFormatter: this.booleanFormatter },
    {
      headerName: 'Actions',
      cellRenderer: this.actionCellRenderer.bind(this)
    }
  ];

  defaultColDef: ColDef = {
    resizable: true,
    sortable: true,
    filter: true
  };

  constructor(
    private dashboardService: DashboardService,
    private renderer: Renderer2,
    private elementRef: ElementRef
  ) {}

  ngOnInit(): void {
    this.loadDashboards();
  }

  loadDashboards(): void {
    this.dashboardService.getDashboards().subscribe(
      (data: Dashboardd[]) => {
        this.tableData = data;
        console.log('Loaded Dashboards', data);
      },
      (error) => {
        console.error('Failed to load dashboards', error);
      }
    );
  }

  dateFormatter(params: any): string {
    return new Date(params.value).toLocaleDateString();
  }

  booleanFormatter(params: any): string {
    return params.value ? 'Yes' : 'No';
  }

  onEditClicked(dashboard: Dashboardd): void {
    console.log('Edit clicked:', dashboard);
    const isInteractive = !!(
      dashboard.model &&
      dashboard.groupBy &&
      dashboard.aggregation &&
      dashboard.aggregationField &&
      dashboard.columnField
    );

    console.log('isInteractive:', isInteractive);
    console.log('Before update:', {
      showBlankDashboard: this.showBlankDashboard,
      showInteractiveDashboard: this.showInteractiveDashboard,
      selectedBlankDashboard: this.selectedBlankDashboard,
      selectedInteractiveDashboard: this.selectedInteractiveDashboard
    });

    if (isInteractive) {
      this.selectedInteractiveDashboard = dashboard;
      this.showInteractiveDashboard = true;
      this.showBlankDashboard = false;
    } else {
      this.selectedBlankDashboard = dashboard;
      this.showBlankDashboard = true;
      this.showInteractiveDashboard = false;
    }

    console.log('After update:', {
      showBlankDashboard: this.showBlankDashboard,
      showInteractiveDashboard: this.showInteractiveDashboard,
      selectedBlankDashboard: this.selectedBlankDashboard,
      selectedInteractiveDashboard: this.selectedInteractiveDashboard
    });
  }

  onDeleteClicked(id: number): void {
    console.log('Delete clicked:', id);
    this.dashboardService.deleteDashboard(id).subscribe(() => {
      this.loadDashboards();
    });
  }

  onDashboardCreated(): void {
    console.log('Dashboard created or updated');
    this.loadDashboards();
    this.onBackToManage();
  }

  onBackToManage(): void {
    this.showBlankDashboard = false;
    this.showInteractiveDashboard = false;
    this.selectedBlankDashboard = undefined;
    this.selectedInteractiveDashboard = undefined;
  }

  onClose(): void {
    console.log('Close clicked');
    this.onBackToManage();
  }

  gridOptions: any = {
    columnDefs: this.columnDefs,
    defaultColDef: this.defaultColDef,
    rowData: this.tableData,
    onGridReady: this.onGridReady.bind(this),
    rowSelection: 'single',
    pagination: true,
    paginationPageSize: 10,
    suppressRowClickSelection: true,
    animateRows: true
  };

  onGridReady(params: any): void {
    params.api.sizeColumnsToFit();
  }

  toggleBlankDashboard(): void {
    this.showBlankDashboard = true;
    this.showInteractiveDashboard = false;
    this.selectedBlankDashboard = undefined;
  }

  toggleInteractiveDashboard(): void {
    this.showInteractiveDashboard = true;
    this.showBlankDashboard = false;
    this.selectedInteractiveDashboard = undefined;
  }

  actionCellRenderer(params: any): HTMLElement {
    const div = this.renderer.createElement('div');
    const editButton = this.renderer.createElement('button');
    const deleteButton = this.renderer.createElement('button');

    this.renderer.addClass(editButton, 'mat-icon-button');
    this.renderer.addClass(deleteButton, 'mat-icon-button');

    const editIcon = this.renderer.createElement('mat-icon');
    const deleteIcon = this.renderer.createElement('mat-icon');

    this.renderer.appendChild(editIcon, this.renderer.createText('edit'));
    this.renderer.appendChild(deleteIcon, this.renderer.createText('delete'));

    this.renderer.appendChild(editButton, editIcon);
    this.renderer.appendChild(deleteButton, deleteIcon);

    this.renderer.appendChild(div, editButton);
    this.renderer.appendChild(div, deleteButton);

    this.renderer.listen(editButton, 'click', () => {
      this.onEditClicked(params.data);
    });

    this.renderer.listen(deleteButton, 'click', () => {
      this.onDeleteClicked(params.data.id);
    });

    return div;
  }
}


blank ts
import { Component, Input, Output, EventEmitter, OnInit } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { CommonModule } from '@angular/common';
import { Router } from '@angular/router';
import { AgChartsModule } from 'ag-charts-angular';
import { AgChartOptions } from 'ag-charts-community';
import { DashboardService } from '../dashboard.service';
import { Dashboardd } from '../dashboard.model';

@Component({
  selector: 'app-blank-dashboard',
  standalone: true,
  imports: [FormsModule, CommonModule, AgChartsModule],
  templateUrl: './blank-dashboard.component.html',
  styleUrls: ['./blank-dashboard.component.css'],
})
export class BlankDashboardComponent implements OnInit {
  @Input() editData?: Dashboardd;
  @Output() dashboardClose = new EventEmitter<void>();

  showModal = true;
  isFullscreen = false;
  currentStep = 0;
  steps = ['Initial', 'Content', 'Layout', 'Review'];
  isEditMode = false;

  formData = {
    name: '',
    description: '',
    isPublic: false,
  };

  selectedChart = 'bar';
  title = '';
  model = '';
  groupBy = '';
  aggregation = '';
  aggregationField = '';

  modelOptions: string[] = [];
  groupByOptions: string[] = [];
  aggregationTypes: string[] = [];
  aggregationFields: string[] = [];

  barChartOptions: AgChartOptions = { data: [], series: [] };
  pieChartOptions: AgChartOptions = {
    data: [],
    title: { text: 'Portfolio Composition' },
    series: [],
  };

  barChartUrl = 'https://isuite-evaluation.dev.echonet/ISUITE/NextGen/assets/barchart.png';
  pieChartUrl = 'https://isuite-evaluation.dev.echonet/ISUITE/NextGen/assets/piechart.png';

  constructor(private router: Router, private dashboardService: DashboardService) {}

  ngOnInit() {
    this.dashboardService.getBlankDashboards().subscribe((data) => {
      this.modelOptions = data.map((item) => item.model).filter((x): x is string => !!x);
      this.groupByOptions = data.map((item) => item.groupBy).filter((x): x is string => !!x);
      this.aggregationTypes = data.map((item) => item.aggregation).filter((x): x is string => !!x);
      this.aggregationFields = data.map((item) => item.aggregationField).filter((x): x is string => !!x);
    });

    if (this.editData) {
      this.isEditMode = true;
      this.formData = {
        name: this.editData.name || '',
        description: this.editData.description || '',
        isPublic: this.editData.isPublic || false,
      };
      this.selectedChart = this.editData.chartType || 'bar';
      this.model = this.editData.model || '';
      this.groupBy = this.editData.groupBy || '';
      this.aggregation = this.editData.aggregation || '';
      this.aggregationField = this.editData.aggregationField || '';
    }
  }

  nextStep() {
    if (this.currentStep === 0 && !this.formData.name.trim()) {
      alert('Please fill in the required "Name" field.');
      return;
    }
    if (this.currentStep < this.steps.length - 1) {
      this.currentStep++;
    }
  }

  prevStep() {
    if (this.currentStep > 0) {
      this.currentStep--;
    }
  }

  setChartType(type: string): void {
    this.selectedChart = type;
    this.title = `Case Queue by Data Element (${type === 'bar' ? 'Bar Chart' : 'Pie Chart'})`;
  }

  toggleFullscreen(): void {
    this.isFullscreen = !this.isFullscreen;
  }

  addCard(): void {
    console.log('Card added:', {
      chartType: this.selectedChart,
      title: this.title,
      model: this.model,
      groupBy: this.groupBy,
      aggregationField: this.aggregationField,
    });
  }

  closeModal(): void {
    this.dashboardClose.emit();
  }

  submitDashboard(): void {
    const dashboardPayload: Dashboardd = {
      name: this.formData.name,
      description: this.formData.description,
      isPublic: this.formData.isPublic,
      chartType: this.selectedChart,
      model: this.model,
      groupBy: this.groupBy,
      aggregation: this.aggregation,
      aggregationField: this.aggregationField,
      createdBy: this.editData?.createdBy || 'h59606',
      createdDate: this.editData?.createdDate || new Date().toISOString(),
      modifiedBy: 'h59606',
      modifiedDate: new Date().toISOString(),
      type: 'blank',
    };

    if (this.isEditMode && this.editData?.id) {
      this.dashboardService.updateDashboard(this.editData.id, dashboardPayload).subscribe({
        next: () => {
          console.log('Dashboard updated');
          this.dashboardClose.emit();
        },
        error: (err) => console.error('Update failed', err),
      });
    } else {
      this.dashboardService.addDashboard(dashboardPayload).subscribe({
        next: () => {
          console.log('Dashboard created');
          this.dashboardClose.emit();
        },
        error: (err) => console.error('Create failed', err),
      });
    }
  }
}

interactive ts
import { Component, EventEmitter, Output, Input, OnInit } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';
import { AgChartsModule } from 'ag-charts-angular';
import { DashboardService } from '../dashboard.service';
import { Dashboardd } from '../dashboard.model';
import { InteractiveDashboard } from '../interactiveDashboard.model';

@Component({
  selector: 'app-interactive-dashboard',
  standalone: true,
  imports: [CommonModule, FormsModule, AgChartsModule],
  templateUrl: './interactive-dashboard.component.html',
  styleUrls: ['./interactive-dashboard.component.css']
})
export class InteractiveDashboardComponent implements OnInit {
  @Input() editData?: Dashboardd;
  @Output() dashboardClose = new EventEmitter<void>();

  showModal = true;
  isFullscreen = false;
  isEditMode = false;
  currentStep = 0;

  steps = ['Initial', 'Content', 'Review'];
  formData = {
    name: '',
    description: '',
    isPublic: true,
  };

  model = '';
  groupBy = '';
  aggregation = '';
  aggregationField = '';
  columnField = '';
  title = '';
  selectedChart = '';

  availableTemplates = ['BarChart', 'PieChart'];
  selectedTemplates: string[] = [];
  dashboards: InteractiveDashboard[] = [];

  barChartUrl = 'https://isuite-evaluation.dev.echonet/ISUITE/NextGen/assets/barchart.png';
  pieChartUrl = 'https://isuite-evaluation.dev.echonet/ISUITE/NextGen/assets/piechart.png';

  constructor(private dashboardService: DashboardService) {}

  ngOnInit(): void {
    this.loadDashboards();

    if (this.editData) {
      this.isEditMode = true;
      this.formData.name = this.editData.name || '';
      this.formData.description = this.editData.description || '';
      this.formData.isPublic = this.editData.isPublic || false;
      this.model = this.editData.model || '';
      this.groupBy = this.editData.groupBy || '';
      this.aggregation = this.editData.aggregation || '';
      this.aggregationField = this.editData.aggregationField || '';
      this.columnField = this.editData.columnField || '';
      this.selectedTemplates = this.editData.selectedTemplates || [];
      this.selectedChart = this.editData.selectedChart || '';
      this.title = this.editData.title || '';
    }
  }

  loadDashboards(): void {
    this.dashboardService.getAllInteractiveDashboards().subscribe(data => {
      this.dashboards = data;
    });
  }

  nextStep(): void {
    if (this.currentStep < this.steps.length - 1) {
      this.currentStep++;
    }
  }

  prevStep(): void {
    if (this.currentStep > 0) {
      this.currentStep--;
    }
  }

  setChartType(chartType: string): void {
    this.selectedChart = chartType;
    this.title = chartType === 'bar'
      ? 'Case Queue by Data Element (Bar Chart)'
      : 'Case Queue by Data Element (Pie Chart)';
    this.toggleTemplate(this.title);
  }

  toggleTemplate(template: string): void {
    if (!this.selectedTemplates.includes(template)) {
      this.selectedTemplates.push(template);
    }
  }

  removeTemplate(template: string): void {
    this.selectedTemplates = this.selectedTemplates.filter(t => t !== template);
  }

  toggleFullscreen(): void {
    this.isFullscreen = !this.isFullscreen;
  }

  addCard(): void {
    this.nextStep();
  }

  closeModal(): void {
    this.dashboardClose.emit();
  }

  submitDashboard(): void {
    const dashboardPayload: Dashboardd = {
      name: this.formData.name,
      description: this.formData.description,
      isPublic: this.formData.isPublic,
      model: this.model,
      groupBy: this.groupBy,
      aggregation: this.aggregation,
      aggregationField: this.aggregationField,
      columnField: this.columnField,
      selectedTemplates: this.selectedTemplates,
      createdBy: this.editData?.createdBy || 'h59606',
      createdDate: this.editData?.createdDate || new Date().toISOString(),
      modifiedBy: 'h59606',
      modifiedDate: new Date().toISOString(),
      chartType: 'interactive',
      type: 'interactive',
    };

    if (this.isEditMode && this.editData?.id) {
      this.dashboardService.updateDashboard(this.editData.id, dashboardPayload).subscribe({
        next: () => {
          console.log('Interactive Dashboard updated');
          this.dashboardClose.emit();
        },
        error: (err) => console.error('Update failed', err),
      });
    } else {
      this.dashboardService.addDashboard(dashboardPayload).subscribe({
        next: () => {
          console.log('Interactive Dashboard created');
          this.dashboardClose.emit();
        },
        error: (err) => console.error('Create failed', err),
      });
    }
  }
}


blank html 
<div class="modal" *ngIf="showModal">
  <div class="modal-content" [class.fullscreen]="isFullscreen">
    <div class="modal-header">
      <h2>{{ isEditMode ? 'Edit Blank Dashboard' : 'New Blank Dashboard' }}</h2>
      <button mat-icon-button (click)="closeModal()">✕</button>
    </div>

    <div class="modal-body">
      <h3>Step {{ currentStep + 1 }}: {{ steps[currentStep] }}</h3>

      <div *ngIf="currentStep === 0">
        <label>Name:</label>
        <input type="text" [(ngModel)]="formData.name" required />

        <label>Description:</label>
        <textarea [(ngModel)]="formData.description"></textarea>

        <label>
          <input type="checkbox" [(ngModel)]="formData.isPublic" />
          Public
        </label>
      </div>

      <div *ngIf="currentStep === 1">
        <label>Model:</label>
        <select [(ngModel)]="model">
          <option *ngFor="let option of modelOptions" [value]="option">{{ option }}</option>
        </select>

        <label>Group By:</label>
        <select [(ngModel)]="groupBy">
          <option *ngFor="let option of groupByOptions" [value]="option">{{ option }}</option>
        </select>

        <label>Aggregation:</label>
        <select [(ngModel)]="aggregation">
          <option *ngFor="let type of aggregationTypes" [value]="type">{{ type }}</option>
        </select>

        <label>Aggregation Field:</label>
        <select [(ngModel)]="aggregationField">
          <option *ngFor="let field of aggregationFields" [value]="field">{{ field }}</option>
        </select>
      </div>

      <div *ngIf="currentStep === 2">
        <div class="chart-selector">
          <img [src]="barChartUrl" alt="Bar Chart" (click)="setChartType('bar')" />
          <img [src]="pieChartUrl" alt="Pie Chart" (click)="setChartType('pie')" />
        </div>
        <p>Selected Chart: {{ selectedChart }}</p>
      </div>

      <div *ngIf="currentStep === 3">
        <p><strong>Name:</strong> {{ formData.name }}</p>
        <p><strong>Description:</strong> {{ formData.description }}</p>
        <p><strong>Chart Type:</strong> {{ selectedChart }}</p>
        <p><strong>Model:</strong> {{ model }}</p>
        <p><strong>Group By:</strong> {{ groupBy }}</p>
        <p><strong>Aggregation:</strong> {{ aggregation }}</p>
        <p><strong>Field:</strong> {{ aggregationField }}</p>
      </div>
    </div>

    <div class="modal-footer">
      <button (click)="prevStep()" [disabled]="currentStep === 0">Back</button>
      <button *ngIf="currentStep < steps.length - 1" (click)="nextStep()">Next</button>
      <button *ngIf="currentStep === steps.length - 1" (click)="submitDashboard()">
        {{ isEditMode ? 'Update' : 'Create' }}
      </button>
    </div>
  </div>
</div>

interactive html
<div class="modal" *ngIf="showModal">
  <div class="modal-content" [class.fullscreen]="isFullscreen">
    <div class="modal-header">
      <h2>{{ isEditMode ? 'Edit Interactive Dashboard' : 'New Interactive Dashboard' }}</h2>
      <button mat-icon-button (click)="closeModal()">✕</button>
    </div>

    <div class="modal-body">
      <h3>Step {{ currentStep + 1 }}: {{ steps[currentStep] }}</h3>

      <div *ngIf="currentStep === 0">
        <label>Name:</label>
        <input type="text" [(ngModel)]="formData.name" required />

        <label>Description:</label>
        <textarea [(ngModel)]="formData.description"></textarea>

        <label>
          <input type="checkbox" [(ngModel)]="formData.isPublic" />
          Public
        </label>
      </div>

      <div *ngIf="currentStep === 1">
        <label>Model:</label>
        <input type="text" [(ngModel)]="model" />

        <label>Group By:</label>
        <input type="text" [(ngModel)]="groupBy" />

        <label>Aggregation:</label>
        <input type="text" [(ngModel)]="aggregation" />

        <label>Aggregation Field:</label>
        <input type="text" [(ngModel)]="aggregationField" />

        <label>Column Field:</label>
        <input type="text" [(ngModel)]="columnField" />

        <div class="template-section">
          <p>Select Templates:</p>
          <div class="template-options">
            <img [src]="barChartUrl" alt="Bar Chart" (click)="toggleTemplate('BarChart')" />
            <img [src]="pieChartUrl" alt="Pie Chart" (click)="toggleTemplate('PieChart')" />
          </div>
          <div class="selected-templates">
            <span *ngFor="let template of selectedTemplates">
              {{ template }}
              <button (click)="removeTemplate(template)">✕</button>
            </span>
          </div>
        </div>
      </div>

      <div *ngIf="currentStep === 2">
        <p><strong>Name:</strong> {{ formData.name }}</p>
        <p><strong>Description:</strong> {{ formData.description }}</p>
        <p><strong>Selected Chart:</strong> {{ selectedChart }}</p>
        <p><strong>Templates:</strong> {{ selectedTemplates.join(', ') }}</p>
        <p><strong>Model:</strong> {{ model }}</p>
        <p><strong>Group By:</strong> {{ groupBy }}</p>
        <p><strong>Aggregation:</strong> {{ aggregation }}</p>
        <p><strong>Field:</strong> {{ aggregationField }}</p>
        <p><strong>Column Field:</strong> {{ columnField }}</p>
      </div>
    </div>

    <div class="modal-footer">
      <button (click)="prevStep()" [disabled]="currentStep === 0">Back</button>
      <button *ngIf="currentStep < steps.length - 1" (click)="nextStep()">Next</button>
      <button *ngIf="currentStep === steps.length - 1" (click)="submitDashboard()">
        {{ isEditMode ? 'Update' : 'Create' }}
      </button>
    </div>
  </div>
</div>