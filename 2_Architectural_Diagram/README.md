# Lab 3 - Component Modelling & Architectural Pattern Selection

**Problem Statement #16 - Remote Patient Vitals Alert & Monitoring App**

## Architecture

The selected architectural style for the system is **Microservices Architecture**.

The system is decomposed into independently manageable services for telemetry
ingestion, vital monitoring and detection, clinical threshold management,
alert generation and escalation, caregiver interaction, and emergency
dispatch integration.

## Deliverables

- [Component Diagram - PNG](Component_Diagram.png)
- [Component Diagram - PDF](Component_Diagram.pdf)
- [PlantUML Source](Component_Diagram.puml)
- [Architecture Justification](Architecture_Justification.pdf)

## Component Overview

The component diagram represents:

- Telemetry Ingestion Service
- Vitals Monitoring & Detection Service
- Clinical Threshold Management Service
- Alert Generation & Escalation Service
- Caregiver Dashboard Service
- Emergency Dispatch Integration Service
- Patient & Alert Data Store

The diagram uses UML provided-interface (ball) and required-interface
(socket) notation to show service interactions and dependencies.

The architecture is based on the requirements and use cases established
in Lab 1 and the functional groupings established during Lab 2.
