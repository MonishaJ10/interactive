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