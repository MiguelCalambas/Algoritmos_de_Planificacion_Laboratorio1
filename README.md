# Algoritmos_de_Planificacion_Laboratorio1

import javax.swing.*;
import javax.swing.table.DefaultTableModel;
import java.awt.BorderLayout;
import java.awt.GridLayout;
import java.util.List;
import java.util.ArrayList;
import java.util.Comparator;
import java.util.Queue;
import java.util.LinkedList;

    public class SimuladorGUI extends JFrame {

        private JTextField txtLlegada, txtRafaga, txtPrioridad, txtQuantum;
        private JComboBox<String> comboAlgoritmo;
        private DefaultTableModel modelo;
        private java.util.List<Proceso> listaProcesos = new ArrayList<>();

        public SimuladorGUI() {

            setTitle("Simulador de Planificación de CPU");
            setSize(950, 500);
            setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
            setLocationRelativeTo(null);
            setLayout(new BorderLayout(10, 10));

            JPanel panelEntrada = new JPanel(new GridLayout(2, 5, 5, 5));

            txtLlegada = new JTextField();
            txtRafaga = new JTextField();
            txtPrioridad = new JTextField();
            txtQuantum = new JTextField();

            comboAlgoritmo = new JComboBox<>(new String[]{
                    "FCFS", "SJF", "Round Robin", "Prioridades"
            });

            panelEntrada.add(new JLabel("Tiempo Llegada"));
            panelEntrada.add(new JLabel("Tiempo Ráfaga"));
            panelEntrada.add(new JLabel("Prioridad"));
            panelEntrada.add(new JLabel("Quantum (RR)"));
            panelEntrada.add(new JLabel("Algoritmo"));

            panelEntrada.add(txtLlegada);
            panelEntrada.add(txtRafaga);
            panelEntrada.add(txtPrioridad);
            panelEntrada.add(txtQuantum);
            panelEntrada.add(comboAlgoritmo);

            add(panelEntrada, BorderLayout.NORTH);

            modelo = new DefaultTableModel();
            modelo.setColumnIdentifiers(new String[]{
                    "Proceso", "Llegada", "Ráfaga", "Prioridad",
                    "Inicio", "Fin", "Espera", "Retorno"
            });

            JTable tabla = new JTable(modelo);
            add(new JScrollPane(tabla), BorderLayout.CENTER);

            JPanel panelBotones = new JPanel();

            JButton btnAgregar = new JButton("Agregar Proceso");
            JButton btnSimular = new JButton("Simular");
            JButton btnLimpiar = new JButton("Limpiar");

            panelBotones.add(btnAgregar);
            panelBotones.add(btnSimular);
            panelBotones.add(btnLimpiar);

            add(panelBotones, BorderLayout.SOUTH);

            btnAgregar.addActionListener(e -> agregarProceso());
            btnSimular.addActionListener(e -> simular());
            btnLimpiar.addActionListener(e -> limpiar());
        }

        private void agregarProceso() {
            try {
                int llegada = Integer.parseInt(txtLlegada.getText());
                int rafaga = Integer.parseInt(txtRafaga.getText());
                int prioridad = Integer.parseInt(txtPrioridad.getText());

                String id = "P" + (listaProcesos.size() + 1);
                Proceso nuevo = new Proceso(id, llegada, rafaga, prioridad);
                listaProcesos.add(nuevo);
                
                modelo.addRow(new Object[]{
                        id, llegada, rafaga, prioridad,
                        "-", "-", "-", "-"
                });

                txtLlegada.setText("");
                txtRafaga.setText("");
                txtPrioridad.setText("");

            } catch (Exception e) {
                JOptionPane.showMessageDialog(this, "Ingrese valores numéricos válidos");
            }
        }

        private void limpiar() {
            listaProcesos.clear();
            modelo.setRowCount(0);
        }

        private void simular() {

            if (listaProcesos.isEmpty()) {
                JOptionPane.showMessageDialog(this, "Agregue procesos primero");
                return;
            }

            modelo.setRowCount(0);

            String algoritmo = comboAlgoritmo.getSelectedItem().toString();

            switch (algoritmo) {
                case "FCFS" -> simularFCFS();
                case "SJF" -> simularSJF();
                case "Round Robin" -> simularRR();
                case "Prioridades" -> simularPrioridades();
            }
        }

        private void simularFCFS() {
            listaProcesos.sort(Comparator.comparingInt(p -> p.llegada));
            calcularNoPreemptivo(listaProcesos);
        }

        private void simularSJF() {
            listaProcesos.sort(Comparator.comparingInt(p -> p.rafaga));
            calcularNoPreemptivo(listaProcesos);
        }

        private void simularPrioridades() {
            listaProcesos.sort(Comparator.comparingInt(p -> p.prioridad));
            calcularNoPreemptivo(listaProcesos);
        }

        private void calcularNoPreemptivo(java.util.List<Proceso> lista) {

            int tiempo = 0;

            for (Proceso p : lista) {

                if (tiempo < p.llegada)
                    tiempo = p.llegada;

                p.inicio = tiempo;
                p.fin = tiempo + p.rafaga;
                p.retorno = p.fin - p.llegada;
                p.espera = p.retorno - p.rafaga;

                tiempo = p.fin;

                modelo.addRow(new Object[]{
                        p.id, p.llegada, p.rafaga, p.prioridad,
                        p.inicio, p.fin, p.espera, p.retorno
                });
            }
        }
        class Proceso {
            String id;
            int llegada, rafaga, prioridad;
            int inicio, fin, espera, retorno;
            int restante;

            public Proceso(String id, int llegada, int rafaga, int prioridad) {
                this.id = id;
                this.llegada = llegada;
                this.rafaga = rafaga;
                this.prioridad = prioridad;
                this.restante = rafaga;
            }

            public Proceso(Proceso p) {
                this.id = p.id;
                this.llegada = p.llegada;
                this.rafaga = p.rafaga;
                this.prioridad = p.prioridad;
                this.restante = p.rafaga;
            }
        }
        // ALGORITMO ROUND ROBIN

        private void simularRR() {

            int quantum;

            try {
                quantum = Integer.parseInt(txtQuantum.getText());
            } catch (Exception e) {
                JOptionPane.showMessageDialog(this, "Ingrese Quantum válido");
                return;
            }

            Queue<Proceso> cola = new LinkedList<>();
            for (Proceso p : listaProcesos)
                cola.add(new Proceso(p));

            int tiempo = 0;

            while (!cola.isEmpty()) {

                Proceso actual = cola.poll();

                if (actual.restante > quantum) {
                    tiempo += quantum;
                    actual.restante -= quantum;
                    cola.add(actual);
                } else {
                    tiempo += actual.restante;
                    actual.fin = tiempo;
                    actual.retorno = actual.fin - actual.llegada;
                    actual.espera = actual.retorno - actual.rafaga;

                    modelo.addRow(new Object[]{
                            actual.id, actual.llegada, actual.rafaga,
                            actual.prioridad, "-", actual.fin,
                            actual.espera, actual.retorno
                    });
                }
            }
        }
        public static void main(String[] args) {
            SwingUtilities.invokeLater(() -> new SimuladorGUI().setVisible(true));
        }

    }
