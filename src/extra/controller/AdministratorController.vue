package com.javaee.onlinehosbackend.extra.controller;

// Importing necessary DTO classes
import com.javaee.onlinehosbackend.dto.AdministratorResponse;
import com.javaee.onlinehosbackend.dto.TreatmentFeedbacksResponse;

// Importing necessary entity classes
import com.javaee.onlinehosbackend.entity.Administrator;

// Importing service class
import com.javaee.onlinehosbackend.service.AdministratorService;

// Importing exception class
import jakarta.persistence.EntityNotFoundException;

// Importing necessary Spring framework classes
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

// Importing utility classes
import java.net.URI;
import java.util.List;
import java.util.stream.Collectors;

@RestController // Marks this class as a REST controller
@RequestMapping("/api/Administrators") // Base URL for all endpoints in this controller
public class AdministratorController {

    @Autowired // Automatically injects the AdministratorService instance
    private AdministratorService administratorService;

    @GetMapping // Handles GET requests to fetch all administrators
    @CrossOrigin // Enables Cross-Origin Resource Sharing (CORS) for this endpoint
    public ResponseEntity<?> getAdministrators() {
        // Fetches all administrators and maps them to AdministratorResponse DTOs
        List<AdministratorResponse> administratorResponseList = administratorService.getAllAdministrators().stream()
                .map(tf -> new AdministratorResponse(
                        tf.getAdministratorId(),
                        tf.getName(),
                        tf.getGender(),
                        tf.getContact(),
                        tf.getPassword()))
                .collect(Collectors.toList());

        // If no administrators found, return 404 response
        if (administratorResponseList.isEmpty()) {
            return ResponseEntity.notFound().build();
        }

        // Returns the list of administrators with a 200 OK response
        return ResponseEntity.ok().body(administratorResponseList);
    }

    @GetMapping("/id") // Handles GET requests to fetch an administrator by ID
    @CrossOrigin // Enables CORS for this endpoint
    public ResponseEntity<?> getAdministratorbyId(@RequestParam String id) {
        // Fetches administrator by ID
        Administrator administrator = administratorService.getAdministratorById(id);

        // If administrator not found, return 400 response
        if (administrator == null) {
            return ResponseEntity.badRequest().body("Not found!");
        }

        // Maps the administrator to AdministratorResponse DTO
        AdministratorResponse administratorResponse = new AdministratorResponse(
                administrator.getAdministratorId(),
                administrator.getName(),
                administrator.getGender(),
                administrator.getContact(),
                administrator.getPassword());

        // Returns the administrator with a 200 OK response
        return ResponseEntity.ok().body(administratorResponse);
    }

    @GetMapping("/name") // Handles GET requests to fetch administrators by name
    @CrossOrigin // Enables CORS for this endpoint
    public ResponseEntity<?> getAdministratorbyName(@RequestParam String name) {
        // Fetches administrators by name and maps them to AdministratorResponse DTOs
        List<AdministratorResponse> administratorResponseList = administratorService.getAdministratorByName(name)
                .stream()
                .map(tf -> new AdministratorResponse(
                        tf.getAdministratorId(),
                        tf.getName(),
                        tf.getGender(),
                        tf.getContact(),
                        tf.getPassword()))
                .collect(Collectors.toList());

        // If no administrators found, return 404 response
        if (administratorResponseList.isEmpty()) {
            return ResponseEntity.notFound().build();
        }

        // Returns the list of administrators with a 200 OK response
        return ResponseEntity.ok().body(administratorResponseList);
    }

    @PostMapping("/add") // Handles POST requests to add a new administrator
    @CrossOrigin // Enables CORS for this endpoint
    public ResponseEntity<?> addAdministrator(@RequestBody Administrator administrator) {
        // Checks if an administrator with the given ID already exists
        if (administratorService.existsById(administrator.getAdministratorId())) {
            return ResponseEntity.badRequest()
                    .body("Administrator with ID " + administrator.getAdministratorId() + " already exists.");
        }

        // Adds the new administrator
        Administrator newAdmin = administratorService.addAdministrator(administrator);

        // Returns the newly added administrator with a 201 Created response
        return ResponseEntity.created(URI.create("/api/administrators/" + newAdmin.getAdministratorId()))
                .body(newAdmin);
    }

    @PutMapping("/update") // Handles PUT requests to update an existing administrator
    @CrossOrigin // Enables CORS for this endpoint
    public ResponseEntity<?> updateAdministrator(@RequestBody Administrator administrator) {
        try {
            // Updates the administrator
            Administrator updatedAdmin = administratorService.updateAdministrator(administrator);

            // Returns the updated administrator with a 200 OK response
            return ResponseEntity.ok(updatedAdmin);
        } catch (EntityNotFoundException e) {
            // If administrator not found, return 404 response
            return ResponseEntity.notFound().build();
        }
    }
}
