package com.javaee.onlinehosbackend.extra.dto;

import com.javaee.onlinehosbackend.dto.PurchaseStatistic;
import com.javaee.onlinehosbackend.dto.SellStatistic;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.math.BigDecimal;
import java.util.List;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class MedicineStatisticsResponse {
    private List<PurchaseStatistic> purchaseStatistics;
    private List<SellStatistic> sellStatistics;


}
