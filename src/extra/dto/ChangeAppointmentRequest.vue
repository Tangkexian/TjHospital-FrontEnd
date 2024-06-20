package com.javaee.onlinehosbackend.extra.dto;

import com.javaee.onlinehosbackend.dto.NewRegistRequest;
import com.javaee.onlinehosbackend.dto.RegistrationRequest;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class ChangeAppointmentRequest {
    private RegistrationRequest old;
    private NewRegistRequest newRegistRequest;
}
