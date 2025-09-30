give curl to test this api by cmd
    @GetMapping("/allemployee")
    public List<Employee> getAllEmployee(){
        return employeeRepository.getAllEmployee();
    }
