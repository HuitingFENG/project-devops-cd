# Use the official Golang image as the base image
FROM golang:latest

# Set the working directory inside the container
WORKDIR /app

# Copy the Go application source code into the container
COPY . .

# Download and install any dependencies
RUN go mod download

# Build the Go application
RUN go build -o project-devops-cd

# Expose the port on which the application will run
EXPOSE 8080

# Command to run the executable
CMD ["./project-devops-cd"]
