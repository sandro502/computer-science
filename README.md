# computer-science
from PyQt5.QtWidgets import (
    QApplication, QWidget, QVBoxLayout, QLineEdit, QTextEdit, QLabel
)
from PyQt5.QtGui import QFont
import sys
import itertools
import re


def extract_variables(expression):
    return sorted(set(re.findall(r'\b[A-Z]\b', expression)))


def generate_truth_table(expression, variables):
    results = []
    for values in itertools.product([True, False], repeat=len(variables)):
        env = dict(zip(variables, values))
        try:
            full_result = eval(expression, {}, env)
            results.append((env, full_result))
        except Exception:
            results.append((env, "Error"))
    return results


def analyze_results(results):
    values = [r[1] for r in results if isinstance(r[1], bool)]
    if all(values):
        return "ვალიდურია "
    elif any(values):
        return "შესრულებადია "
    else:
        return "არაშესრულებადია "


class LogicApp(QWidget):
    def __init__(self):
        super().__init__()
        self.setWindowTitle("ლოგიკური ფორმულის ანალიზი")
        self.setGeometry(100, 100, 1200, 700)

        layout = QVBoxLayout()

        self.label = QLabel(" შეიყვანეთ ლოგიკური ფორმულა:")
        self.label.setFont(QFont("Arial", 16))
        layout.addWidget(self.label)

        self.input_line = QLineEdit()
        self.input_line.setFont(QFont("Consolas", 14))
        self.input_line.returnPressed.connect(self.evaluate_expression)
        layout.addWidget(self.input_line)

        self.output_text = QTextEdit()
        self.output_text.setReadOnly(True)
        self.output_text.setFont(QFont("Consolas", 12))
        layout.addWidget(self.output_text)

        self.setLayout(layout)

    def evaluate_expression(self):
        expression = self.input_line.text().strip()
        variables = extract_variables(expression)
        results = generate_truth_table(expression, variables)

        col_width = max(12, len(expression) + 2)
        headers = variables + [expression]
        output = " | ".join(h.center(col_width) for h in headers) + "\n"
        output += "-+-".join(["-" * col_width for _ in headers]) + "\n"

        for env, final in results:
            row = [str(env[v]).center(col_width) for v in variables]
            row.append(str(final).center(col_width))
            output += " | ".join(row) + "\n"

        output += "\n დასკვნა: " + analyze_results(results)
        self.output_text.setText(output)


if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = LogicApp()
    window.show()
    sys.exit(app.exec_())
